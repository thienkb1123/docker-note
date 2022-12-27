```Docker
FROM golang:1.18-alpine AS builder

ENV GO111MODULE=on \
    CGO_ENABLED=0  \
    GOARCH="amd64" \
    GOOS=linux

RUN apk update && apk add make git pkgconfig gcc g++ bash musl-dev

WORKDIR /app
COPY . .
RUN go mod download && go build -tags musl -ldflags "-extldflags '-static' -s -w" -o myapp main.go


FROM scratch

WORKDIR /app
COPY --from=builder /app/myapp ./

CMD ["./myapp"]
```

Bật `CGO_ENABLED=1` nếu sử dụng những thư viện hay package viết bằng C/C++ như `librdkafka`
Tham số của `go build` và `ldflags`

Có thể thêm tham số `-a` (ví dụ `go build -a main.go`) để rebuild tất cả packages

`-extldflags '-static'` để nói với Go truyền cho bộ external linker tham số `-static`. Có thể sử dụng thêm `-linkmode external` để nói với Go luôn sử dụng external linker (ví dụ `-ldflags "-linkmode external -extldflags '-static'"`)

`-s` loại bỏ khỏi binary phần symbol table và thông tin debug

`-w` loại bỏ thêm khỏi binary phần DWARF symbol table

### Build distroless image

`FROM scratch` là image rỗng của docker image. Build từ image này sẽ không có những file hay program của nhân linux (như bash, sh, ...). Nếu cần sử dụng thì chép từ image `builder` sang.

Ưu điểm:

- Kích thước image nhỏ
- Loại bỏ những file/chương trình hệ thống không sử dụng trong docker image
Hạn chế những lỗi security, khi 1 container bị tấn công cũng không chạy được những lệnh của hệ thống

Khuyết điểm:

- Đòi hỏi phải hiểu sâu những thư viện hệ thống cần thiết liên quan tới app đang build
- Thời gian build image sẽ lâu hơn

`alpine` sử dụng `musl libc` (khoảng 5.6mb). Nếu build app dạng shared/dynamic, thì chỉ cần chép thêm thư viện musl theo.

> COPY --from=builder /lib/ld-musl-x86_64.so.1 /lib/

SSL
Nếu service có sử dụng SSL (ví dụ như gọi API bên ngoài qua https), thì phải chép certificates

> COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt


Thay vì image `scratch` có thể sử dụng `gcr.io/distroless/static` (khoảng 2.36mb)

Tham khảo https://github.com/GoogleContainerTools/distroless

From: A Cường =))