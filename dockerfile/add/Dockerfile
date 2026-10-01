FROM alpine:latest

RUN mkdir hello

# Kita gunakan match pattern untuk menambahkan semua file txt ke dalam folder hello
ADD text/*.txt hello

CMD ["cat", "hello/world.txt"]
