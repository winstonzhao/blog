
---

title:  Average Frame of a Movie
tags:
- C++

---

To prepare for my upcoming job, I've been searching for C++ projects.  I saw this post on [reddit](https://www.reddit.com/r/dataisbeautiful/comments/3rb8zi/the_average_color_of_every_frame_of_a_given_movie/) around a year ago and I thought,  *"I could do (copy) that"*. 

### Results
Avengers Trailer
![](/images/avengers.png)

Sicko Mode - Travis Scott
![](/images/sicko_mode.png)

L$D - ASAP Rocky
![](/images/lsd.png)

California Gurls - Katy Perry
![](/images/california_gurls.png)

The Search - NF
![](/images/the_search.png)

### Splitting an Image into Frames
To manually extract the frames of a media file, you would have to account for the compression algorithms to calculate the pixel values of each frame. This would be extremely complex, so I just googled instead and found [this](https://www.bugcodemaster.com/article/extract-images-frame-frame-video-file-using-ffmpeg).

We can use the following `ffmpeg` command that does what we want and will create a bunch of bitmap files that correspond to the frames of the video.
```bash
ffmpeg -i video.webm thumb%d.bmp -hide_banner
```
Bitmap is simple format without any compression and thus is a good format to use for this project.

### Reading Images into Memory
I copied the following code from [stackoverflow](https://stackoverflow.com/questions/9296059/read-pixel-value-in-bmp-file).

```cpp
FILE *f =  fopen(filename.c_str(),  "rb");
unsigned  char  info[54];
fread(info,  sizeof(unsigned  char),  54, f); // read the 54-byte header
// extract image height and width from header
int width, height;
std::memcpy(&width, info +  18,  sizeof(int));
std::memcpy(&height, info +  22,  sizeof(int));
int size =  3  * width *  abs(height);
unsigned  char  *data =  new  unsigned  char[size]; // allocate 3 bytes per pixel
fread(data,  sizeof(unsigned  char), size, f); // read the rest of the data at once
fclose(f);
```
After finishing the project, I decided to actually learn how this code worked.
```cpp
FILE *f =  fopen(filename.c_str(),  "rb");
```
`fopen` returns a stream of the data of the file, however by default `fopen` is used for text files. However, by specifying **rb** as the mode, we let `fopen` know that we want to **r**ead  **b**inary.
```cpp
fread(info,  sizeof(unsigned  char),  54, f);
```
If we look at the bitmap file format information on [wikipedia](https://en.wikipedia.org/wiki/BMP_file_format).  We can assume that the total length of file headers are  54 bytes.

There are various versions of headers that the bitmap file may have, and for a more robust approach we would need to take the other versions into consideration. However, for this project it suffices to to assume the `BITMAPINFOHEADER` version.

```cpp
std::memcpy(&width, info +  18,  sizeof(int));
std::memcpy(&height, info +  22,  sizeof(int));
int size =  3  * width *  abs(height);
```

After extracting the width and the height of the image we can calculate the size of the pixel data in the image. Each pixel is represented by three 8bit integers representing the respective `RGB` values. 

```cpp
unsigned  char  *data =  new  unsigned  char[size];  // allocate 3 bytes per pixel 
fread(data,  sizeof(unsigned  char), size, f);  // read the rest of the data at once 
fclose(f);
```
We can now copy in the image data and then close the file.

### Writing Images to Disk
We can write a struct to represent the headers.
```cpp
#pragma  pack(2)
struct  BitmapFileHeader
{
	char  header[2]{'B', 'M'};
	int32_t fileSize;
	int32_t reserved{0};
	int32_t dataOffset;
};

struct  BitmapInfoHeader
{
	int32_t headerSize{40};
	int32_t width;
	int32_t height;
	int16_t planes{1};
	int16_t bitsPerPixel{24};
	int32_t compression{0};
	int32_t dataSize{0};
	int32_t horizontalResolution{2400};
	int32_t verticalResolution{2400};
	int32_t colors{0};
	int32_t importantColors{0};
};
```
The previously linked [wikipedia](https://en.wikipedia.org/wiki/BMP_file_format) page about has more information on what these fields actually do. We use the `BITMAPINFOHEADER` version of the info header.

Now to actually write the image to disk.

```cpp
BitmapInfoHeader infoHeader;
BitmapFileHeader fileHeader;
fileHeader.fileSize = sizeof(BitmapFileHeader) + sizeof(BitmapInfoHeader) + m_width * m_height * 3;
fileHeader.dataOffset = sizeof(BitmapFileHeader) + sizeof(BitmapInfoHeader);
infoHeader.width = m_width;
infoHeader.height = m_height
std::ofstream file;
file.open(filename, std::ios::out | std::ios::binary);
file.write((char *)&fileHeader, sizeof(fileHeader));
file.write((char *)&infoHeader, sizeof(infoHeader));
file.write((char *)m_data.get(), m_width * m_height * 3);
file.close();
return  true;
```

Most of the values that needed to be specified in the header are constant do not need to be modified.

However, we still need to write the file size into the file header, and this can only be known at the time of writing.

```cpp
fileHeader.fileSize = sizeof(BitmapFileHeader) + sizeof(BitmapInfoHeader) + m_width * m_height * 3;
```

We write data offset, width and height similarly.

```cpp
fileHeader.dataOffset = sizeof(BitmapFileHeader) + sizeof(BitmapInfoHeader);
infoHeader.width = m_width;
infoHeader.height = m_height
```

We open the file using a `ofstream`

```cpp
std::ofstream file;
file.open(filename, std::ios::out | std::ios::binary);
```

We specify that we will be writing `out` to the file and we expect to be writing in `binary`.

Now, write both headers and the data into a file, before closing it.  We cast the data to `char *` since the `ofstream` expects to be writing text.

```cpp
file.write((char *)&fileHeader, sizeof(fileHeader));
file.write((char *)&infoHeader, sizeof(infoHeader));
file.write((char *)m_data.get(), m_width * m_height * 3);
file.close();
```

### Finding the Mean Color
This part was fairly easy, we can simply sum up the RGB values and divide them by the total amount of pixels to get the average color. 

```cpp
RGB ret(0, 0, 0);
int n = 0;
for (int x = 0; x < m_width; x += 10)
{
	for (int y = 0; y < m_height; y += 10)
	{
		auto rgb = getRGB(x, y);
		ret = ret + rgb;
		n++;
	}
}
return ret / n;
```
I won't go into detail about how the RGB class or `getRGB()` works, as more details are be found in the full source code.

Here's some results I got from the Avenger's Trailer.

![](/images/avengers.png)

### Using Mode Color
While, the results with the mean color were as expected. I wanted to see if I could get more vibrant spectrum using the mode color instead.  

We would have to first cluster the colors (go from 24bit RGB to a lower bit value). Otherwise, it would be unlikely that enough of any specific 24bit RGB value would occur to yield any meaningful results. I found out that the "mode color" was basically always black, so I decided to filter out the black color from the results and got the following picture.

![](/images/avengers_mode.png)

It didn't look as a good as the mean color, so I left the project here.
