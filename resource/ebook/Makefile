BUILD = build
BOOKNAME = how_to_write_with_markdown
TITLE =
METADATA = metadata.xml
PT=chap/
CHAPTERS = $(PT)ch01.md $(PT)ch02.md
TOC = --toc --toc-depth=4
COVER_IMAGE = img/cover.jpg
LATEX_CLASS = report



all: book

book: html pdf

clean:
	rm -r $(BUILD)


html: $(BUILD)/html/$(BOOKNAME).html

pdf: report  book

$(BUILD)/html/$(BOOKNAME).html: $(CHAPTERS)
	mkdir -p $(BUILD)/html
	pandoc $(TOC) --standalone --to=html5 -o $@ $^

report: $(BUILD)/pdf/$(BOOKNAME)_report.pdf
$(BUILD)/pdf/$(BOOKNAME)_report.pdf: $(TITLE) $(CHAPTERS)
	mkdir -p $(BUILD)/pdf
	pandoc $(TOC) --latex-engine=xelatex --template=temp.tex -V documentclass=report -o $@ $^  \
  -V	classoption=oneside \
	-V fontsize=12pt		\
  -V	mainfont=Ubuntu 	-V sansfont=Ubuntu	 -V	monofont=Ubuntu -V monofont=Ubuntu  -V mathfont=Ubuntu  \
	-V linkcolor=blue		-V filecolor=red -V citecolor=yellow -V urlcolor=gray -V toccolor=blue	\
	-V toc -V toc-depth=4	 -V pagestyle=headings \
	-V margin-left=1.7cm -V margin-right=1.7cm -V margin-top=2cm -V margin-bottom=2cm

book: $(BUILD)/pdf/$(BOOKNAME)_book.pdf
$(BUILD)/pdf/$(BOOKNAME)_book.pdf:$(TITLE) $(CHAPTERS)
		mkdir -p $(BUILD)/pdf
		pandoc -N $(TOC) --latex-engine=xelatex --template=temp.tex -V documentclass=book -o $@ $^  \
	  -V	classoption=oneside \
		-V fontsize=12pt		\
	  -V	mainfont=Ubuntu 	-V sansfont=Ubuntu	 -V	monofont=Ubuntu -V monofont=Ubuntu  -V mathfont=Ubuntu  \
		-V linkcolor=blue		-V filecolor=red -V citecolor=yellow -V urlcolor=gray -V toccolor=blue	\
		 -V pagestyle=headings \
		-V margin-left=1.5cm -V margin-right=1.5cm -V margin-top=2cm -V margin-bottom=2cm


beamer:$(BUILD)/pdf/$(BOOKNAME)_beamer.pdf

$(BUILD)/pdf/$(BOOKNAME)_beamer.pdf: $(TITLE) $(CHAPTERS)
	mkdir -p $(BUILD)/pdf
	pandoc --latex-engine=xelatex  -o $@ $^ -t beamer

.PHONY: all book clean  html pdf
