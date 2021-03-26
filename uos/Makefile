pdf:
	pandoc --pdf-engine=xelatex \
		--highlight-style tango \
		-V geometry:"top=2cm, bottom=1.5cm, left=2cm, right=2cm" \
		-V urlcolor=NavyBlue \
		--toc \
		--toc-depth 5 \
		-f markdown-implicit_figures \
		./uos-deploy.md -o uos-deploy.pdf
