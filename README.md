## PDFCropService
PDFCropService is a workflow for the macOS Finder `Services` menu. It crops vector or bitmap 
images saved as PDF files from the print dialog, for example. It is useful when you want to insert
high-quality graphics into a, say, LaTeX, Word or Pages document and your graphics app only 
offers saving graphics as full pages from the print dialog. PDFCropService
automatically trims all white margins, so that any picture can easily be imported into a text
document.

## Setup

1. Install [Homebrew](https://brew.sh). You can do that with `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
2. Install `ps2eps` which does all the work (convert single page PDFs to EPS and calculate
the bounding box): `brew install ps2eps`
3. Double click the Automator service `Crop PDF Graphic`. You can decide whether to open it in 
Automator or to install it. If you choose to install it, it will be placed inside the  
`Services` folder in your user's `Library` folder.
4. After installation the service is available in the `Services` context menu when you
click on a PDF.

## Usage

Using Finder, right-click on a PDF (or several PDFs) to open the context menu, navigate 
to `Services` and select `Crop PDF Graphic`. The original PDF will remain untouched, the cropped 
file will be marked with `_cropped`. You can try it with the included `test.pdf`:

`test.pdf` -> `test_cropped.pdf`
 
 
## Info
 
Inside the Automator document there is a bash script:
 
```bash
for f in "$@"
do
	# Edit given file path
	name=$( basename "$f" .pdf) # name of file without path and without extension
	dir=$( dirname "$f" ) # name of directory of file
	cd "$dir" # Make it the working directory

	# Add brew directories to PATH variable, since ps2eps is a Pearl script which uses Ghostscript
	# Just providing the complete path to ps2eps is not sufficient
	PATH=$PATH:/usr/local/bin:/opt/local/bin

	# Convert to Postscript format, taking care of whitespace characters
	eval pdf2ps "'$f'"

	# Trim all whitespace for ps2eps
	name_nowhitespace="$(echo -e "${name}" | tr -d '[:space:]')"
	mv "$name".ps "$name_nowhitespace".ps

	# Calculate and set bounding box, --loose expands the original tight bounding box by one point in each direction
	ps2eps --loose "$name_nowhitespace".ps

	# Convert back to PDF using Ghostscript, important option is "-dEPSCrop"
	gs -q -dNOPAUSE -dBATCH -dDOINTERPOLATE -dUseFlateCompression=true -sDEVICE=pdfwrite -r1200 -dEPSCrop -sOutputFile="$name"_cropped.pdf -f "$name_nowhitespace".eps

	# Remove temporary files
	rm "$name_nowhitespace".ps
	rm "$name_nowhitespace".eps
done
```

If you happen to have [MacTeX](http://tug.org/mactex/) or a similar LaTeX version installed, 
you can edit the script and replace `pdf2ps`, `ps2eps` and `gs` with a call to [`pdfcrop`](https://www.ctan.org/pkg/pdfcrop?).
