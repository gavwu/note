### Frame

[Frame Format](https://www.rfc-editor.org/rfc/rfc7540#section-4.1)

根据Frame的定义

> All frames begin with a fixed 9-octet header followed by a variable-length payload.

```
    +-----------------------------------------------+
    |                 Length (24)                   |
    +---------------+---------------+---------------+
    |   Type (8)    |   Flags (8)   |
    +-+-------------+---------------+-------------------------------+
    |R|                 Stream Identifier (31)                      |
    +=+=============================================================+
    |                   Frame Payload (0...)                      ...
    +---------------------------------------------------------------+
```

Frame整体可以分为
- FrameHeader
- FramePayload

### FrameHeader

Go对FrameHeader有对应的一个结构体

```go
// A FrameHeader is the 9 byte header of all HTTP/2 frames.
//
// See https://httpwg.org/specs/rfc7540.html#FrameHeader
type FrameHeader struct {
	valid bool // caller can access []byte fields in the Frame

	// Type is the 1 byte frame type. There are ten standard frame
	// types, but extension frame types may be written by WriteRawFrame
	// and will be returned by ReadFrame (as UnknownFrame).
	Type FrameType

	// Flags are the 1 byte of 8 potential bit flags per frame.
	// They are specific to the frame type.
	Flags Flags

	// Length is the length of the frame, not including the 9 byte header.
	// The maximum size is one byte less than 16MB (uint24), but only
	// frames up to 16KB are allowed without peer agreement.
	Length uint32

	// StreamID is which stream this frame is for. Certain frames
	// are not stream-specific, in which case this field is 0.
	StreamID uint32
}

func readFrameHeader(buf []byte, r io.Reader) (FrameHeader, error) {
	_, err := io.ReadFull(r, buf[:frameHeaderLen])
	if err != nil {
		return FrameHeader{}, err
	}
	return FrameHeader{
		// byte 0~2
		Length:   (uint32(buf[0])<<16 | uint32(buf[1])<<8 | uint32(buf[2])),
		// byte 3
		Type:     FrameType(buf[3]),
		// byte 4
		Flags:    Flags(buf[4]),
		// byte 5~8
		// 0-bit always 0
		StreamID: binary.BigEndian.Uint32(buf[5:]) & (1<<31 - 1),
		valid:    true,
	}, nil
}
```

从这里的结构不难与上面FramHeader的图形描述对应上

### FramePayload

FramePayload根据不同Frame有不同的定义

#### Data Frame

```
    +---------------+
    |Pad Length? (8)|
    +---------------+-----------------------------------------------+
    |                            Data (*)                         ...
    +---------------------------------------------------------------+
    |                           Padding (*)                       ...
    +---------------------------------------------------------------+
```

```go
// WriteDataPadded writes a DATA frame with optional padding.
//
// If pad is nil, the padding bit is not sent.
// The length of pad must not exceed 255 bytes.
// The bytes of pad must all be zero, unless f.AllowIllegalWrites is set.
//
// It will perform exactly one Write to the underlying Writer.
// It is the caller's responsibility not to violate the maximum frame size
// and to not call other Write methods concurrently.
func (f *Framer) WriteDataPadded(streamID uint32, endStream bool, data, pad []byte) error {
	if !validStreamID(streamID) && !f.AllowIllegalWrites {
		return errStreamID
	}
	if len(pad) > 0 {
		if len(pad) > 255 {
			return errPadLength
		}
		if !f.AllowIllegalWrites {
			for _, b := range pad {
				if b != 0 {
					// "Padding octets MUST be set to zero when sending."
					return errPadBytes
				}
			}
		}
	}
	var flags Flags
	if endStream {
		flags |= FlagDataEndStream
	}
	if pad != nil {
		flags |= FlagDataPadded
	}
	f.startWrite(FrameData, flags, streamID)
	if pad != nil {
		f.wbuf = append(f.wbuf, byte(len(pad)))
	}
	f.wbuf = append(f.wbuf, data...)
	f.wbuf = append(f.wbuf, pad...)
	return f.endWrite()
}
```

1. 校验Pad是否合理
2. 写FrameHeader
3. 写Data Frame Payload

#### Header Frame

[Header Frame](https://www.rfc-editor.org/rfc/rfc7540#section-6.2)
```go
// WriteHeaders writes a single HEADERS frame.
//
// This is a low-level header writing method. Encoding headers and
// splitting them into any necessary CONTINUATION frames is handled
// elsewhere.
//
// It will perform exactly one Write to the underlying Writer.
// It is the caller's responsibility to not call other Write methods concurrently.
func (f *Framer) WriteHeaders(p HeadersFrameParam) error {
	if !validStreamID(p.StreamID) && !f.AllowIllegalWrites {
		return errStreamID
	}
	var flags Flags
	if p.PadLength != 0 {
		flags |= FlagHeadersPadded
	}
	if p.EndStream {
		flags |= FlagHeadersEndStream
	}
	if p.EndHeaders {
		flags |= FlagHeadersEndHeaders
	}
	if !p.Priority.IsZero() {
		flags |= FlagHeadersPriority
	}
	f.startWrite(FrameHeaders, flags, p.StreamID)
	if p.PadLength != 0 {
		f.writeByte(p.PadLength)
	}
	if !p.Priority.IsZero() {
		v := p.Priority.StreamDep
		if !validStreamIDOrZero(v) && !f.AllowIllegalWrites {
			return errDepStreamID
		}
		if p.Priority.Exclusive {
			v |= 1 << 31
		}
		f.writeUint32(v)
		f.writeByte(p.Priority.Weight)
	}
	f.wbuf = append(f.wbuf, p.BlockFragment...)
	f.wbuf = append(f.wbuf, padZeros[:p.PadLength]...)
	return f.endWrite()
}


func (f *Framer) startWrite(ftype FrameType, flags Flags, streamID uint32) {
	// Write the FrameHeader.
	f.wbuf = append(f.wbuf[:0],
		0, // 3 bytes of length, filled in in endWrite
		0,
		0,
		byte(ftype),
		byte(flags),
		byte(streamID>>24),
		byte(streamID>>16),
		byte(streamID>>8),
		byte(streamID))
}

func (f *Framer) endWrite() error {
	// Now that we know the final size, fill in the FrameHeader in
	// the space previously reserved for it. Abuse append.
	length := len(f.wbuf) - frameHeaderLen
	if length >= (1 << 24) {
		return ErrFrameTooLarge
	}
	_ = append(f.wbuf[:0],
		byte(length>>16),
		byte(length>>8),
		byte(length))
	if f.logWrites {
		f.logWrite()
	}

	n, err := f.w.Write(f.wbuf)
	if err == nil && n != len(f.wbuf) {
		err = io.ErrShortWrite
	}
	return err
}
```
