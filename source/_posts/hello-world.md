
---

title:  Average Frame of a Movie

---

To prepare for my upcoming job, I've been searching for C++ projects.  I saw this post on [reddit](https://www.reddit.com/r/dataisbeautiful/comments/3rb8zi/the_average_color_of_every_frame_of_a_given_movie/) around a year ago and I thought,  *"I could do (copy) that"*. 

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
return  std::make_unique<Bitmap>(Bitmap(height, width, data));
```
After finishing the project, I decided to actually learn how this code worked.
```cpp
FILE *f =  fopen(filename.c_str(),  "rb");
```
`fopen` returns a stream of the data of the file, however by default `fopen` is used for text files. However, by specifying **rb** as the mode, we let `fopen` know that we want to **r**ead  **b**inary.
```cpp
fread(info,  sizeof(unsigned  char),  54, f);
```
If we look at the bitmap file format information on [wikipedia](https://en.wikipedia.org/wiki/BMP_file_format)

### Writing Images to Disk
### Finding the Average Color

