# keep it 
clean:
	rm -rf build
	
build:
	mkdir -p build
	pandoc --defaults=defaults.yaml -F pandoc-crossref --citeproc -o build/result.docx src/chapter_1/*.md src/chapter_2/*.md