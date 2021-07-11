# PDFCropService
PDFCropService is a workflow for the macOS Finder `Services` menu. It crops vector or bitmap 
images saved as PDF files from the print dialog, for example. It is useful when you want to insert
high-quality graphics into a, say, LaTeX, Word or Pages document and your graphics app only 
offers saving graphics as full pages from the print dialog. PDFCropService
automatically trims all white margins, so that any picture can easily be imported into a text
document.

## Setup

1. Download ZIP file from this GitHub page.

2. Install [Homebrew](https://brew.sh). You can do that with `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

3. Install `ps2eps` which does all the work (convert single page PDFs to EPS and calculate
the bounding box): `brew install ps2eps`

4. Double click the Automator service `Crop PDF Graphic`. You can decide whether to open it in 
Automator or to install it. If you choose to install it, it will be placed inside the `Services` folder in your user's `Library` folder.

or

4. If you have [MacTeX](http://tug.org/mactex/) installed, double click the Automator service `Crop PDF with pdfcrop`. You can decide whether to open it in 
Automator or to install it. If you choose to install it, it will be placed inside the `Services` folder in your user's `Library` folder. This script version uses [`pdfcrop`](https://www.ctan.org/pkg/pdfcrop?) from the MacTeX installation.

After installation the service is available in the `Services` context menu when you
click on a PDF.

## Usage

Using Finder, right-click on a PDF (or several PDFs) to open the context menu, navigate 
to `Services` and select `Crop PDF Graphic` or `Crop PDF with pdfcrop`. The original PDF will remain untouched, the cropped file will be marked with `_cropped`. You can try it with the included `test.pdf`:

`test.pdf` -> `test_cropped.pdf`
 
 
## Info
 
Inside the Automator document `Crop PDF Graphic` there is a `bash` script:
 
```bash
for f in "$@"
do
	# Edit given file path
	name=$( basename "$f" .pdf) # name of file without path and without extension
	dir=$( dirname "$f" ) # name of directory of file
	cd "$dir" # Make it the working directory

	# Add brew directories to PATH variable, since ps2eps is a Pearl script which uses Ghostscript
	# Just providing the complete path to ps2eps is not sufficient
	PATH=$PATH:/usr/local/bin:/opt/local/bin:/opt/homebrew/bin

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

## Troubleshooting

If you have [MacTeX](http://tug.org/mactex/) installed, both script versions will probably fail with an error from ghostscript ("Can't find initialization file gs_init.ps"). If this happens, you can issue `brew link --overwrite ghostscript` to force the usage of homebrew's version of ghostscript.

`pdfcrop` from the MacTeX package also only works if you do that.
