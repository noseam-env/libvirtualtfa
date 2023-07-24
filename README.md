# libvirtualtfa (VirtualTFA)

This library is an implementation of a virtual write and read in parts archive format TFA (Transfer File Archive). This technology allows you to comfortably stream archives over the network, as well as read streams of archives without buffering. It is needed to completely eliminate temporary files and accelerate writing and reading. The virtualization essentially reads file data directly from the disk into the final buffer, but in between it includes its headers using pointer logic.

Example:

```c++
int main(int argc, char *argv[]) {
    VirtualTfaArchive *archive = virtual_tfa_archive_new();
    
    for (int i = 1; i < argc; ++i) {
        char* argument = argv[i];
        //
        // Here you need to create object that
        // implements VirtualFile class
        //
        VirtualTfaEntry *entry = virtual_tfa_entry_new(virtualFile);
        virtual_tfa_archive_add(archive, entry);
    }
    
    VirtualTfaWriter tfaWriter = *new VirtualTfaWriter(archive, nullptr);
    std::cout << "Archive size: " << std::to_string(tfaWriter.calcSize()) << std::endl;
    
    std::ofstream fileOut("output.tfa", std::ios::binary);
    if (fileOut.is_open()) {
        char buffer[8192];
        
        std::size_t bytesWritten;
        do {
            bytesWritten = tfaWriter.writeTo(buffer, sizeof(buffer));
            if (bytesWritten > 0) {
                fileOut.write(buffer, bytesWritten);
            }
        } while (bytesWritten > 0);

        fileOut.close();

        std::cout << "Done!" << std::endl;
    } else {
        std::cout << "File is closed" << std::endl;
    }
}
```


## Specification

### Header

Size: `48 bytes`

| Field    | Size | Absolute Pos | Description                                     |
|----------|------|--------------|-------------------------------------------------|
| magic    | 4    | 0-3          | magic field, value `tfa\0`                      |
| version  | 1    | 4            | tfa version, currently `\0`                     |
| typeflag | 1    | 5            | currently unused                                |
| unused   | 4    | 6-9          | reserved bytes                                  |
| mode     | 8    | 10-17        | file permissions in bitset<9>                   |
| ctime    | 8    | 18-25        | file creation UNIX time (uint64_t)              |
| mtime    | 8    | 26-33        | file last modification UNIX time (uint64_t)         |
| namesize | 6    | 34-39        | size of file name in bytes (including last `\0`) |
| filesize | 8    | 40-47        | total file size (uint64_t)              |

### Structure

| Header   | File Name       | File Data       | Header   | File Name         |     |
|----------|-----------------|-----------------|----------|-------------------|-----|
| 48 bytes | header.namesize | header.filesize | 48 bytes | header.namesize | ... |


## License

The library is licensed under the [MIT License](https://opensource.org/license/mit/):

Copyright (C) 2023 Michael Neonov <two.nelonn@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Authors

- **Michael Neonov** ([email](mailto:two.nelonn@gmail.com), [github](https://github.com/Nelonn))
