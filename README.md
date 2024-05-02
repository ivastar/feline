# feline

### get data cubus
```
wget http://martinwendt.de/cube.fits
```

## Run Preprocessing, Feline and Plot-Program in Python2.7 virtual environment

### Create virtual environment with anaconda:
```
conda create -n felineenv python=2.7
```
### activate environment:
```
conda activate felineenv
```
### install all necessary packages:

```
conda install numpy
```
```
conda install scipy
```
```
conda install matplotlib
```
```
conda install astropy
```
```
pip install mpdaf pyfits scikit-image ref_index
```
## In preprocess dir:
### apply median filter to 'flat' out the data (emission lines remain):
```
python median-filter-cube.py ../cube.fits --signalHDU=1 --varHDU=2 --num_cpu=6 --width=151 --output=med_filt.fits
```
### filter the data cube with a 'typical line' template in spatial dimension first:
```
python lsd_cc_spatial_mask.py --input=med_filt.fits --SHDU=0 --NHDU=1 --threads=4 --gaussian --lambda0=7050 -p0=0.7 --output=spatial_cc.fits
```
### filter the data cube with a 'typical line' template in spectral dimension:
```
python lsd_cc_spectral.py --input=spatial_cc.fits --threads=2 --FWHM=250 --SHDU=0 --NHDU=1 --output=spectral_cc.fits
```
### filter the data cube with a 'typical line' template in spectral dimension:
```
python s2n-cube.py --input=spectral_cc.fits --output=s2n_v250.fits --clobber
```
### construct a signal-to-noise cube:
```
cd ..
```
```
cp preprocess/s2n_v250.fits s2n_v250.fits
```
```
cp preprocess/med_filt.fits med_filt.fits
```

## here the actual tool starts:

### optional masking plot and fast cache access
```
python combination.py cube.fits s2n_v250.fits
```

### build and run the main Feline code (C/OpenMP):
```
make
```
```
./feline.bin 0 1.9 20 7
```
### detect actual objects and translate into physical properties (redshift, line list) sorted by significance:
```
python detect_objects.py s2n_v250.fits > catalog.txt
```
```
sort -rn -k5 catalog.txt > sorted_catalog.txt
```
### create comprehensive human readable plots for each detection:
```
python create_final_plots.py cube.fits s2n_v250.fits sorted_catalog.txt med_filt.fits J0014m0028
```

### create a PDF file containing all plotss:
```
python create_pdf.py
```