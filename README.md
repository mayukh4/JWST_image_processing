# JWST Image Processing

A Python toolkit for processing raw James Webb Space Telescope (JWST) data into beautiful, colorized composite images.

![my_jwst_image](https://github.com/user-attachments/assets/38135e6a-b3a8-45e5-85f0-fe6a35f7d37a)

## Overview

This project provides tools to convert raw FITS files from the James Webb Space Telescope into stunning visual images. It handles the entire image processing pipeline including:

- Loading FITS files with accurate metadata extraction
- Contrast enhancement and dynamic range compression
- Image alignment and reprojection
- Wavelength-based colorization
- Layer blending for final composite creation

The toolkit is designed for both astronomers and enthusiasts, with detailed comments and explanations throughout the code.

## Features

- **Complete Processing Pipeline**: Process raw JWST FITS files into publication-quality images
- **Customizable Colorization**: Define custom colors for each filter or use wavelength-based automatic assignment
- **Memory-Efficient**: Handles large JWST images (100+ megapixels) through slice-based processing
- **Scientific Accuracy**: Preserves important relationships between different wavelengths
- **User-Friendly**: Well-documented code with sensible defaults for non-experts

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/jwst-image-processing.git
cd jwst-image-processing

# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

## Dependencies

- numpy
- matplotlib
- astropy
- reproject
- scikit-image
- Pillow (PIL)

## Usage

### Basic Usage

The quickest way to process JWST images:

```python
from jwst_processor import process_fits_folder

# Process a folder of FITS files into a composite image
process_fits_folder(
    fits_folder="path/to/fits_files",
    output_image="output/cosmic_cliffs.jpg",
    layers_folder="layers"  # Optional, to save individual filter images
)
```

### Custom Color Assignment

To customize the colors for each layer:

```python
from jwst_processor import combine_layers

# Combine preprocessed layers with custom colors
combine_layers(
    layers_folder="layers",
    output_image="output/cosmic_cliffs_custom.jpg",
    colors_file="colors.txt"  # Optional color specification file
)
```

#### Colors File Format

Each line specifies the filepath and HSV (Hue, Saturation, Value) for a layer:

```
"layers/JW02731_NIRCAM-F090W.png" (240, 87, 100)
"layers/JW02731_NIRCAM-F200W.png" (200, 67, 100)
"layers/JW02731_NIRCAM-F335M.png" (150, 100, 100)
"layers/JW02731_NIRCAM-F444W.png" (65, 90, 100)
"layers/JW02731_NIRCAM-F470N.png" (0, 100, 100)
```

You can also specify ranges for random values (e.g., `200-240` for hue).

## Command Line Interface

The package includes two main scripts:

1. `fits-to-image.py`: Converts FITS files to a composite image

```bash
python fits-to-image.py fits/ cosmic_cliffs.jpg layers/
```

2. `combine-layers.py`: Combines layer images with custom colors

```bash
python combine-layers.py layers/ cosmic_cliffs_custom.jpg cosmic_cliffs.txt
```

## Examples

### Cosmic Cliffs (Carina Nebula)

```python
# Process the Cosmic Cliffs (NGC 3324) in the Carina Nebula
process_fits_folder(
    fits_folder="carina_nebula_fits",
    output_image="cosmic_cliffs.jpg",
    layers_folder="carina_layers"
)
```

### Southern Ring Nebula

```python
# Process the Southern Ring Nebula
process_fits_folder(
    fits_folder="southern_ring_fits",
    output_image="southern_ring.jpg",
    layers_folder="southern_ring_layers"
)
```

## Getting JWST Data

All JWST data becomes publicly available after a 12-month proprietary period. You can download raw data from the [MAST Portal](https://mast.stsci.edu/portal/Mashup/Clients/Mast/Portal.html).

### Steps to download data:

1. Go to the [MAST Portal](https://mast.stsci.edu/portal/Mashup/Clients/Mast/Portal.html)
2. Click "Advanced Search"
3. Filter by Mission: "JWST" and Instrument: "NIRCAM" and/or "MIRI"
4. Enter an object name (e.g., "Carina Nebula" or "NGC 3324")
5. Select the files ending in `_i2d.fits` (Level 3 calibrated images)
6. Download the data

## How It Works

### 1. Loading and Calibrating FITS Files

The `WebbsterFITS` class handles loading FITS files and extracting metadata:

```python
# Creates an object with the data and metadata
fits_obj = WebbsterFITS("jwst_image.fits")
```

### 2. Contrast Enhancement

The `adjust_contrast()` method applies calibration and dynamic range compression:

```python
# Adjust contrast to make dim features visible
fits_obj.adjust_contrast()
```

### 3. Image Alignment

The `reproject()` method aligns images from different filters:

```python
# Align all images to the reference image
fits_obj.reproject(reference_image)
```

### 4. Colorization

The `WebbsterLayer` class handles colorization:

```python
# Create a layer from the FITS object
layer = WebbsterLayer.fromFITS(fits_obj)

# Colorize the layer
layer.colorize()
```

### 5. Blending

The `screen_blend_multiple()` function combines all colorized layers:

```python
# Blend all layers into a final image
final_image = screen_blend_multiple([layer.color_image for layer in layers])
```

## Implementation Details

### Color Assignment Logic

Colors are assigned based on wavelength using an S-curve adjustment to avoid excessive green:

```python
# Map wavelength to hue
wl_prop = (wavelength - min_wavelength) / (max_wavelength - min_wavelength)
wl_prop_adj = 1 / (1 + (wl_prop / (1 - wl_prop)) ** -2)
hue = max_hue - wl_prop_adj * (max_hue - min_hue)
```

### Screen Blending

Layers are combined using screen blending, which mimics how light combines:

```python
# Screen blend formula
blended = 255 - (((255 - image1) * (255 - image2)) // 255)
```
![jwst_colorized_blending](https://github.com/user-attachments/assets/0995c631-7f9e-4afd-8a6c-926df7ef21ff)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- This project is inspired by the incredible work of the JWST team
- JWST data is provided by [STScI](https://www.stsci.edu/) through the [MAST Portal](https://mast.stsci.edu/)
- Special thanks to the Astropy and reproject developers

## Contact

- Mayukh Bagchi bagchimayukh4@gmail.com
- Blog Post: [Link to your detailed tutorial]()
- Video Tutorial: [Link to your YouTube video]()
