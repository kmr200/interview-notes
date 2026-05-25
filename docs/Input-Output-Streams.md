# Input/Output Streams

## IO vs NIO

| Feature           | Java IO (Input-Output)                                                                                                 | Java NIO (New/Non-blocking IO)                                                                                                |
|-------------------|------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| Orientation       | Stream-oriented                                                                                                        | Buffer-oriented                                                                                                               |
| Data Processing   | Reads/writes one or more bytes sequentially from/to a stream                                                           | Reads/writes data into a buffer, allowing more flexibility                                                                    |
| Blocking Behavior | Blocking - when calling `read()` or `write()` from `java.io.*`, the thread is blocked until the operation is completed | Non-blocking - the thread can request data from a channel and receive only what is available (or nothing if no data is ready) |
| Flexibility       | No caching; cannot move randomly within the data stream                                                                | Uses buffers, allowing for better control of data movement                                                                    |
| Efficiency        | Each stream requires a separate thread for handling I/O                                                                | A single thread can monitor multiple channels using selectors, making it more efficient for handling multiple connections     |
| Selectors         | Not available                                                                                                          | Uses selectors, enabling one thread to handle multiple channels (ideal for scalable applications)                             |

In IO, when a thread performs I/O operations, it must wait until the data is fully read or written, preventing it from doing anything else. In contrast, NIO allows a thread to continue working on other tasks while waiting for data to be available. This makes Java NIO particularly useful for handling high-performance, scalable network applications, such as web servers, where multiple connections need to be managed efficiently.

### Key Features of Java NIO

| Feature              | Description                                                                                                                                                                                                                                                                                                    |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Channels & Selectors | NIO supports various channels, which act as abstractions over lower-level file system objects (e.g., memory-mapped files, file locks). Channels are non-blocking, allowing Java to provide tools like selectors for monitoring multiple channels. Selectors enable handling multiple data streams efficiently. |
| Buffers              | Instead of working with individual bytes or character streams, NIO uses buffers for all primitive wrapper classes (except `Boolean`). The abstract class `Buffer` provides methods like `clear()`, `flip()`, and `mark()`. Subclasses offer methods for setting and retrieving data efficiently.               |
| Encoding & Decoding  | Introduced encoders and decoders for converting between byte arrays and Unicode characters (useful for handling text data).                                                                                                                                                                                    |

---

## Channels in Java NIO

Channels are logical (not physical) portals for input/output operations, abstracting lower-level system objects like memory-mapped files and file locks. They provide an efficient way to read/write data asynchronously using buffers.

- When **writing** data, the data is first stored in a buffer, which is then passed to a channel.
- When **reading** data, the channel sends it to a pre-allocated buffer for processing.

Channels behave like pipelines, allowing efficient data transfer between byte buffers and external data sources.

**Advantages of Channels:**
- Minimal overhead in accessing OS-level I/O services.
- Supports non-blocking operations, improving efficiency.
- Works well with Selectors, making it ideal for scalable applications (e.g., servers handling multiple connections).

---

## Java I/O Packages

- `java.io` - Contains traditional blocking I/O classes.
- `java.nio` - Provides non-blocking I/O (NIO) for improved performance.
- `java.util.zip` - Contains classes for working with compressed data streams (e.g., `GZIPInputStream`, `ZipInputStream`).

### Main Stream Classes

| Type              | Classes                                       |
|-------------------|-----------------------------------------------|
| Byte Streams      | `java.io.InputStream`, `java.io.OutputStream` |
| Character Streams | `java.io.Reader`, `java.io.Writer`            |

---

## InputStream Subclasses

`InputStream` is an abstract class that defines the behavior of input byte streams.

| Subclass                                 | Description                                                                                                                       |
|------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `BufferedInputStream`                    | Buffered input stream that improves performance by reducing the number of read operations.                                        |
| `ByteArrayInputStream`                   | Reads data from a byte array in memory instead of a file or other source.                                                         |
| `DataInputStream`                        | Reads primitive Java data types (e.g., `int`, `double`, `boolean`) from an input stream.                                          |
| `FileInputStream`                        | Reads data from a file on disk.                                                                                                   |
| `FilterInputStream`                      | Abstract class that provides a wrapper for other input streams, allowing additional processing.                                   |
| `ObjectInputStream`                      | Reads serialized objects from an input stream.                                                                                    |
| `StringBufferInputStream` *(Deprecated)* | Converts a `String` into an input stream. Use `ByteArrayInputStream` instead.                                                     |
| `PipedInputStream`                       | Used in inter-thread communication, where one thread writes to a `PipedOutputStream` and another reads from a `PipedInputStream`. |
| `PushbackInputStream`                    | Allows unreading bytes, meaning data can be "peeked" before reading further.                                                      |
| `SequenceInputStream`                    | Merges two or more input streams into a single stream.                                                                            |

### PushbackInputStream

`PushbackInputStream` is a buffered input stream that allows "unreading" bytes. It enables a program to peek ahead into the input stream before actually consuming the data. This is useful in parsing scenarios, where you may need to check an incoming byte before deciding how to process it.

Key method: `unread(int b)` - pushes a byte back into the stream so that it can be read again.

### SequenceInputStream

`SequenceInputStream` combines multiple `InputStream` objects into a single input stream. It reads from the first stream until it is exhausted, then switches to the next, and so on.

Constructor options:
- `SequenceInputStream(InputStream s1, InputStream s2)` - merges two input streams.
- `SequenceInputStream(Enumeration<? extends InputStream> streams)` - merges multiple input streams from an `Enumeration`.

### DataInputStream - Reading Primitive Data Types

`DataInputStream` allows reading primitive Java data types from an input byte stream.

| Method          | Reads                |
|-----------------|----------------------|
| `readBoolean()` | 1-byte boolean       |
| `readByte()`    | 1-byte byte          |
| `readChar()`    | 2-byte char          |
| `readDouble()`  | 8-byte double        |
| `readFloat()`   | 4-byte float         |
| `readInt()`     | 4-byte int           |
| `readLong()`    | 8-byte long          |
| `readShort()`   | 2-byte short         |
| `readUTF()`     | UTF-8 encoded String |

---

## OutputStream Subclasses

| Class                   | Description                                                                                        |
|-------------------------|----------------------------------------------------------------------------------------------------|
| `OutputStream`          | Abstract base class for all byte output streams.                                                   |
| `BufferedOutputStream`  | Buffered output stream that reduces the number of writes to improve performance.                   |
| `ByteArrayOutputStream` | Stores data in an internal byte array instead of writing to a file or socket.                      |
| `DataOutputStream`      | Writes Java primitive types (`int`, `double`, `boolean`, etc.) in a platform-independent way.      |
| `FileOutputStream`      | Writes data directly to a file on disk.                                                            |
| `FilterOutputStream`    | Abstract decorator class for modifying output streams (e.g., adding buffering or data conversion). |
| `PrintStream`           | Supports formatted text output with `print()` and `println()`, commonly used for console logging.  |
| `ObjectOutputStream`    | Writes serialized Java objects to a stream, allowing them to be saved and restored later.          |
| `PipedOutputStream`     | Used with `PipedInputStream` to enable communication between threads.                              |

---

## Reader Subclasses

| Class               | Description                                                                       |
|---------------------|-----------------------------------------------------------------------------------|
| `Reader`            | Abstract base class for character input streams.                                  |
| `BufferedReader`    | Buffered input stream that improves reading efficiency and supports `readLine()`. |
| `CharArrayReader`   | Reads characters from a character array.                                          |
| `FileReader`        | Reads characters from a file.                                                     |
| `FilterReader`      | Abstract class for creating reader decorators (extensions of `Reader`).           |
| `InputStreamReader` | Converts byte streams (`InputStream`) into character streams (`Reader`).          |
| `LineNumberReader`  | Tracks the current line number while reading.                                     |
| `PipedReader`       | Used with `PipedWriter` for inter-thread communication.                           |
| `PushbackReader`    | Allows characters to be "pushed back" into the stream for re-reading.             |
| `StringReader`      | Reads characters from a `String`.                                                 |

---

## Writer Subclasses

| Class                | Description                                                               |
|----------------------|---------------------------------------------------------------------------|
| `Writer`             | Abstract base class for character output streams.                         |
| `BufferedWriter`     | Buffered output stream that improves writing efficiency.                  |
| `CharArrayWriter`    | Writes characters to a character array.                                   |
| `FileWriter`         | Writes characters to a file.                                              |
| `FilterWriter`       | Abstract class for creating writer decorators (extensions of `Writer`).   |
| `OutputStreamWriter` | Converts character streams (`Writer`) into byte streams (`OutputStream`). |
| `PipedWriter`        | Used with `PipedReader` for inter-thread communication.                   |
| `PrintWriter`        | Supports formatted text output with `print()` and `println()` methods.    |
| `StringWriter`       | Writes characters to a `String`.                                          |

---

## Comparisons

### PrintWriter vs. PrintStream

| Feature            | PrintWriter                                                            | PrintStream                                                    |
|--------------------|------------------------------------------------------------------------|----------------------------------------------------------------|
| Handles            | Character (text) output                                                | Byte output                                                    |
| Automatic Flushing | Optional, controlled with `flush()`                                    | Flushes automatically after `print()` or `println()`           |
| Unicode Handling   | Uses proper character encoding                                         | Works with raw bytes; may not handle Unicode correctly         |
| Error Handling     | Does not throw exceptions; requires `checkError()` to check for errors | Also requires `checkError()` but may handle errors differently |
| Usage              | Preferred for text output, writing characters, and formatted text      | Used for writing raw byte data, such as binary files           |

### InputStream, OutputStream, Reader, and Writer

| Type           | Purpose                   | Data Type Handled           | Common Use Cases                              |
|----------------|---------------------------|-----------------------------|-----------------------------------------------|
| `InputStream`  | Byte-oriented input       | Reads raw bytes             | Reading files, network streams, binary data   |
| `OutputStream` | Byte-oriented output      | Writes raw bytes            | Writing files, network streams, binary data   |
| `Reader`       | Character-oriented input  | Reads characters (Unicode)  | Reading text files, parsing strings           |
| `Writer`       | Character-oriented output | Writes characters (Unicode) | Writing text files, logging, formatted output |

---

## Buffered Classes for Faster I/O

| Class                                              | Description                                                    |
|----------------------------------------------------|----------------------------------------------------------------|
| `BufferedInputStream(InputStream in)`              | Buffered byte input stream for improving read performance.     |
| `BufferedInputStream(InputStream in, int size)`    | Buffered byte input stream with a specified buffer size.       |
| `BufferedOutputStream(OutputStream out)`           | Buffered byte output stream for improving write performance.   |
| `BufferedOutputStream(OutputStream out, int size)` | Buffered byte output stream with a specified buffer size.      |
| `BufferedReader(Reader r)`                         | Buffered character input stream for efficient text reading.    |
| `BufferedReader(Reader in, int sz)`                | Buffered character input stream with a specified buffer size.  |
| `BufferedWriter(Writer out)`                       | Buffered character output stream for efficient text writing.   |
| `BufferedWriter(Writer out, int sz)`               | Buffered character output stream with a specified buffer size. |

---

## The File Class

The `File` class represents files and directories, allowing operations such as creation, deletion, and metadata retrieval.

### Common Methods

| Method                        | Description                                                                              |
|-------------------------------|------------------------------------------------------------------------------------------|
| `boolean createNewFile()`     | Attempts to create a new file.                                                           |
| `boolean delete()`            | Attempts to delete a file or directory.                                                  |
| `boolean mkdir()`             | Creates a new directory.                                                                 |
| `boolean renameTo(File dest)` | Renames a file or directory.                                                             |
| `boolean exists()`            | Checks if a file or directory exists.                                                    |
| `String getAbsolutePath()`    | Returns the absolute path of the file or directory.                                      |
| `String getName()`            | Returns the name of the file or directory.                                               |
| `String getParent()`          | Returns the name of the parent directory.                                                |
| `boolean isDirectory()`       | Checks if the path is a directory.                                                       |
| `boolean isFile()`            | Checks if the path is a file.                                                            |
| `boolean isHidden()`          | Checks if the file or directory is hidden.                                               |
| `long length()`               | Returns the size of the file in bytes.                                                   |
| `long lastModified()`         | Returns the last modified timestamp of the file or directory.                            |
| `String[] list()`             | Returns an array of filenames in a directory.                                            |
| `File[] listFiles()`          | Returns an array of `File` objects representing files and subdirectories in a directory. |

### FileFilter Interface

`FileFilter` is used to test whether a file matches certain conditions. It contains a single method - `boolean accept(File pathName)` - which you implement to define your own criteria.

```java
public boolean accept(final File file) {
    return file.exists() && file.isDirectory();
}
```

### Selecting Files by Criteria (e.g., File Extension)

`File.listFiles()` returns an array of `File` objects from a directory. You can pass a `FileFilter` to filter files based on a condition such as extension or size.

```java
File directory = new File("path/to/directory");
FileFilter filter = new FileFilter() {
    public boolean accept(File file) {
        return file.getName().endsWith(".txt");
    }
};
File[] txtFiles = directory.listFiles(filter);
```

---

## RandomAccessFile

`RandomAccessFile` allows reading and writing data at arbitrary positions within a file, unlike sequential streams.

### Key Methods

| Method                      | Description                                    |
|-----------------------------|------------------------------------------------|
| `getFilePointer()`          | Returns the current position in the file.      |
| `seek(long pos)`            | Moves the file pointer to a specific position. |
| `length()`                  | Returns the size of the file.                  |
| `setLength(long newLength)` | Changes the file's size.                       |
| `skipBytes(int n)`          | Skips a specified number of bytes.             |
| `getChannel()`              | Returns the file's associated `FileChannel`.   |

Also supports: `read()`, `readInt()`, `write()`, `writeInt()`, and other read/write methods.

### Access Modes

| Mode    | Description                                                                                  |
|---------|----------------------------------------------------------------------------------------------|
| `"r"`   | Read-only. Throws `IOException` on write attempts.                                           |
| `"rw"`  | Read and write. Creates the file if it doesn't exist.                                        |
| `"rws"` | Read and write with synchronization to physical storage on every change (data and metadata). |
| `"rwd"` | Like `"rws"`, but only synchronizes data changes, not metadata.                              |

---

## Compressed Data Streams

### Compression Output Streams

- `DeflaterOutputStream` - Compresses data using the deflated algorithm.
- `ZipOutputStream` - Compresses data in the ZIP format.
- `GZIPOutputStream` - Compresses data using the GZIP format.

### Compression Input Streams

- `InflaterInputStream` - Decompresses data using the deflate algorithm.
- `ZipInputStream` - Decompresses data in the ZIP format.
- `GZIPInputStream` - Decompresses data in the GZIP format.

---

## Redirecting Standard I/O Streams

The `System` class provides these static methods for redirecting standard streams:

| Method                           | Effect                           |
|----------------------------------|----------------------------------|
| `System.setIn(InputStream in)`   | Redirects standard input.        |
| `System.setOut(PrintStream out)` | Redirects standard output.       |
| `System.setErr(PrintStream err)` | Redirects standard error output. |

---

## File Paths

### Path Separators

- Windows uses `\` as a path separator.
- Linux/Unix uses `/` as a path separator.
- In Java, use `File.separator` to get the separator for the current OS.

### Absolute vs. Relative Paths

**Absolute path:** A full path independent of the current working directory, always starting from the root (e.g., `/home/user/file.txt` or `C:\Users\file.txt`).

**Relative path:** A path relative to the current working directory (e.g., `docs/file.txt`).

---

## Symbolic Links (Symlinks)

A symbolic link (symlink) is a special file that contains a path to another file or directory, acting as a pointer or shortcut.

- Can link to any object, including another symlink, file, or directory.
- Unlike hard links, symbolic links can span different file systems.
- Can link to non-existent files (resulting in an error when accessed).

**Creating a symlink in Linux:**

```bash
ln -s /path/to/original/file /path/to/symlink
```