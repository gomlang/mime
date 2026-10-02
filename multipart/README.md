# Multipart

`Writer[W: Write]` streams MIME multipart bodies with a caller-supplied
validated boundary. `begin_part(textproto::Headers)` serializes one header
block, `write_body_chunk` emits arbitrary binary data, and `finish` writes the
closing delimiter without closing the sink. Body data that would form a
delimiter is rejected, including matches split across input chunks.

`Reader[R: Read]` accepts a bounded preamble, exposes part headers through
`next_part`, and streams each body through `read_body_chunk`. A body must be
drained until it returns zero before advancing to the next part. Header
continuations join with one space; malformed framing and source failures are
terminal. Boundary lines accept trailing ASCII space/tab transport padding as required by
[RFC 2046 section 5.1.1](https://www.rfc-editor.org/rfc/rfc2046.html#section-5.1.1),
including the initial boundary, intervening boundaries and the closing delimiter.
The closing delimiter may end with CRLF or at EOF after padding. Boundary line
content, including marker and padding, is bounded by `max_line_bytes`.
Within a part body, oversized potential boundaries fail while padding is read;
false boundary prefixes remain body data within those bounds. Before the first
part, the reader buffers a complete line to distinguish preamble from a delimiter,
then checks a recognized delimiter against `max_line_bytes`. That initial line
buffer is bounded by the greater of `max_preamble_bytes`, `max_line_bytes` and
the closing marker length, plus two CRLF bytes; `max_wire_bytes` also applies.
The reader leaves any epilogue unread.

`Limits` bounds wire bytes, preamble, headers, line and value bytes, part bytes,
aggregate body bytes, and part count. The package preserves duplicate headers
and byte-valued metadata; applications choose their own form-data field and
filename policy. It does not perform filesystem extraction or transfer-codec
decoding.

The writer rejects boundary collisions with SP/HTAB transport padding, including collisions split across body chunks and a padded marker at the end of a part.
