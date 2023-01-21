<p align="center">
    <img src=https://cdn0.iconfinder.com/data/icons/landscape-collection/383/mountain_river-512.png width=138/>
</p>

<h1 align="center">Greenfield Image Format</h1>

<p align="center"><strong>A very simple Uniform Quantizated RGB Image Format</strong></p>

<table align="center">
    <thead>
        <tr>
            <th align="center">Project</th>
            <th align="center">Description</th>
            <th align="center">Crate</th>
            <th align="center">Docs</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><a target="_blank" href="https://github.com/Tomcat-42/greenfield">greenfield</td>
            <td align="center">A Rust greenfield library</td>
            <td align="center">
                <a href="https://crates.io/greenfield" target="_blank">
                <img src="https://img.shields.io/crates/v/greenfield"></a>
            </td>
            <td>
                <a href="https://docs.rs/greenfield" target="_blank">
                <img src="https://img.shields.io/docsrs/greenfield"></a>
            </td>
        </tr>
         <tr>
            <td align="center"><a target="_blank" href="https://github.com/Tomcat-42/greenriver">greenriver</td>
            <td align="center">A Rust CLI greenfield application</td>
            <td align="center">
                <a href="https://crates.io/greenriver" target="_blank">
                <img src="https://img.shields.io/crates/v/greenriver"></a>
            </td>
            <td>
                <a href="https://docs.rs/greenfield" target="_blank">
                <img src="https://img.shields.io/docsrs/greenriver"></a>
            </td>
        </tr>
    </tbody>
</table>

## The greenfield image format

The greenfield image format is a simple **2D array of colors**, prefixed with
the **width** and **height** of the image, and a **quantization information**.
All of the field are stored in **big endian** format.

- The first 64 bits(a full u64) are the greenfield magic value, used to identify
  the file as a greenfield image (`b"grnfld42"`).
- The next 16 bits (a full u64) bits are the width of the image.
- The next 16 bits (a full u64) bits are the height of the image.
- The next 12 bits are the quantization information tuple (see
  \[`quantization`\]). A qunatization tuple is in the form:
  `(bits_r, bits_g, bits_b)`, where each value is the number of bits used to
  store the respective color component.
- The remaining bits are the image color data, in row-major order. Each color
  has (bits_r + bits_g + bits_b) bits. So, for example, if the quantization
  tuple is `(5, 6, 5)`, then each color is 16 bits. To get all the colors, you
  must read (width _height)_ (bits_r + bits_g + bits_b) bits.

## Format on Disk

```text
╔════════════════════════════╤══════════════════════════════════════════════════════════╗
║            Bits            │                      Description                         ║
╠════════════════════════════╪══════════════════════════════════════════════════════════╣
║             64             │      b"grnfld42": Magic value (0x47524E464C443432)       ║
╟────────────────────────────┼──────────────────────────────────────────────────────────╢
║             16             │                   u32: Image width                       ║
╟────────────────────────────┼──────────────────────────────────────────────────────────╢
║             16             │                   u32: Image height                      ║
╟────────────────────────────┼──────────────────────────────────────────────────────────╢
║             12             │      (bits_r, bits_g, bits_b): Quantization tuple        ║
╟────────────────────────────┼──────────────────────────────────────────────────────────╢
║      width * height *      │       [RGB]: (bits_r + bits_g + bits_r) per pixel        ║
║ (bits_r + bits_g + bits_b) │                       row-major                          ║
╚════════════════════════════╧══════════════════════════════════════════════════════════╝
```

## Color Quantization

Rgb quantization is the process of reducing the color space size of an image
with the objective of reducing the size of the image in disk. This is done by
grouping similar colors in the space and referring to the group instead of a
particular color.

Greenfield images uses a quantization technique know as **Uniform
Quantization**. In uniform quantization, we divide each component of the color
space in equal intervals, indexing each interval with a number, and then we
assign a color to each interval (usually the mean of this interval), in the end,
we just store on disk this index, instead of the color. With that, we can reduce
the number of bits needed to store each component of a color. Once an image has
been loaded from disk, we can find the respective color for each index (the mean
of an interval represented by that index) and reconstruct the image.

For example, if we have a 24-bit RGB image, we can reduce the number of bits
needed to store each component to 5 bits, reducing the number of bits needed to
store each pixel from 24 to 15. Reducing each component to 5 bits, we now have
2^5 = 32 possible values for each component. Each distinct value is the mean of
the interval in the RGB color space.
