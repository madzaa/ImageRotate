```
package com.rotate;

/*
Read in the BMP file as a stream in bytes.
Manipulate the bytes so that it would affect the picture
Shift the bytes so that the image would rotate 90 degrees
*/

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Paths;

public class Main {
public static void main(String[] args) throws IOException {
String path = "image-rotate/teapot.bmp";
byte[] bytes = Files.readAllBytes(Paths.get(path));
int height = getHeaderValue(bytes, 0x16, 0x1a);
int width = getHeaderValue(bytes, 0x12, 0x16);
int offset = getHeaderValue(bytes, 0x0a, 0x0e);
createEmptyBitMap(bytes, height, width, offset);
}

    private static void createEmptyBitMap(byte[] bytes, int height, int width, int offset) {
        byte[] emptyBitMapArray = new byte[bytes.length];
        
        // copy the header info to the new bmp file
        
        for (int i = 0x0; i < 0x36; i++) {
            byte aByte = bytes[i];
            emptyBitMapArray[i] = aByte;
        }

        // 420 pixels in width and height, each pixel being 3 bytes
        
        int counter = offset;
        for (int ty = 0; ty < height; ty++) {
            for (int tx = 0; tx < width; tx++) {
                int sx = height - ty - 1; // formula to calculate the source x  when rotated
                int sourceIndex = offset + 3 * (tx * width + sx); 
                copyBytes(bytes, emptyBitMapArray, counter, sourceIndex); 
                counter += 3; // increment the counter by 3 (bytes) to move to the next pixel
            }
        }
        try (FileOutputStream outputStream = new FileOutputStream("rotated.bmp")) {
            outputStream.write(emptyBitMapArray);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    // sequence to copy the bytes from the source image
    private static void copyBytes(byte[] bytes, byte[] emptyBitMapArray, int counter, int sourceIndex) {
        emptyBitMapArray[counter] = bytes[sourceIndex];
        emptyBitMapArray[counter + 1] = bytes[sourceIndex + 1]; 
        emptyBitMapArray[counter + 2] = bytes[sourceIndex + 2];
    }

    private static int getHeaderValue(byte[] bytes, int start, int end) {
        int header = 0;
        for (int i = start; i < end; i++) {
            byte aByte = bytes[i];
            header |= getShiftedHex(aByte, i - bytes.length);
        }
        return header;
    }

        // Using Javas under the hood shifting logic instead of creating a counter.
        // When doing bitwise shifts in Java it automatically applies the AND 0x1f which makes it a positive shift number
        // And we only use the lowest 5 order bits in shifting
        
    private static int getShiftedHex(byte aByte, int counter) {
        int n = aByte & 0xff;
        return n << (counter * 8);
    }
    
    // Utility method for checking hex value in java as it automatically gets converted into a signed int when used as a variable
    
    private static String getHex(int aByte) {
        return Integer.toHexString(aByte);
    }
}
```