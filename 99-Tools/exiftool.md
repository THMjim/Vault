# exiftool
- used to look at metadata of files
- Pen-200 12.3.1, found author of PDF, used to craft email with EVIL attachment 
```bash
exiftool -a -u info.pdf
```
| Component | Value | Description |
| :---: | :---: | :--- |
| **`exiftool`** | *(Command)* | The name of the utility. **ExifTool** is a powerful, platform-independent command-line application used for reading, writing, and editing metadata in a wide variety of file formats, including images, audio, video, and PDF documents. |
| **`-a`** | *(Switch)* | The **Allow Duplicates** option. This tells ExifTool to display **all metadata tags**, even those with the same name. By default, ExifTool suppresses duplicate tags unless they hold different values. |
| **`-u`** | *(Switch)* | The **Unknown** tag option. This tells ExifTool to also display metadata tags that it **does not recognize** (unknown tags). This is often used for forensic or in-depth analysis. |
| **`info.pdf`** | *(Argument)* | The **Target File** path. This specifies the PDF file from which ExifTool should extract the metadata. |
