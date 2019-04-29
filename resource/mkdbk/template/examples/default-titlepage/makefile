

all: book

book:bk.pdf

bk.pdf: default-titlepage.md
	pandoc    -o $@  --from markdown $^  --template=eisvogel.latex  --listings
