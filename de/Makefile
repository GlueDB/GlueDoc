all: Glue.pdf

Glue.pdf: Glue.tex Makefile
	lualatex Glue.tex
	biber Glue
	makeindex Glue.idx
	lualatex Glue.tex
	lualatex Glue.tex

Glue.tex: *.md Glue.latex Referenzen.bib
	pandoc -o Glue.tex *.md --template Glue.latex --chapter \
	-f markdown+auto_identifiers+implicit_header_references+inline_notes+multiline_tables+implicit_figures+link_attributes \
	--biblatex --toc --bibliography=Referenzen.bib --listings --standalone

.PHONY: clean all

clean:
	sudo rm -r `biber --cache`
	rm -f Glue.aux
	rm -f Glue.bbl
	rm -f Glue.bcf
	rm -f Glue.blg
	rm -f Glue.idx
	rm -f Glue.log
	rm -f Glue.out
	rm -f Glue.run.xml
	rm -f Glue.tex
	rm -f Glue.pdf

