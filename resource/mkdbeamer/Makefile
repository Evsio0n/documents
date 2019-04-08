BUILD = build
BOOKNAME = how_to_write_with_markdown
TITLE = ovew_pool_time
PT=chap/
CHAPTERS = $(PT)bm.md




all: beamer



beamer:$(TITLE)_beamer.pdf

$(TITLE)_beamer.pdf: $(CHAPTERS)
	pandoc --latex-engine=xelatex --template=temp.tex  -o $@ $^ -t beamer -V theme:Warsaw -V toc-title
