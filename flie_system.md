# File-System

## File-System Interface

### File concept

The operating system provides a uniform logical view of stored information. The operating system abstracts from the physical properties of its storage devices to define a logical storage unit, the **file**. Files are mapped by the operating system onto physical devices. These storage devices are usually nonvolatile, so the contents are persistent between system reboots.

In general, a file is a sequence of bits, bytes, or records, the meaning of which is defined by the file's creator and user. Commonly, files represent programs and data.

The information in a file is defined by its creator. Many different types of information may be stored on a file - source or executable programs, text data and so on. A file has a certain defined structure, which depends on its type. A **text file** is a sequence of characters of characters organiced into lines. A **source file** is a series of code sections the loader can bring into memory and execute.

#### File Attributes

A file is named and is referred to by its name. When a file is named, it becomes independent of the process, the user, and even the system that created it. The typical file atributes are:

- Name

- Indentifie

- Type

- Location

- Size

- Protection

- Timestamps and user identification

The information about files is kept in the directory structure, which resides on the same device as the files themselves

#### File Operations

- **Creating a file**. Two steps are needed in order to create a file. First, space in the file system must be found for the file. Second, an entry for the new file must be made in a directory.

- **Opening a file**. Rather than have all file operations specify a file name, causing the operating systems to evaluate the name, check acces permissions and so on, all operations except create and delete require a file `open()` first

- **Writing a file**. System call specifying both the open file handle and the information to be written to the file. **Write pointer** to the location in the file where the next write is to take place if it is sequential.

- **Reading a file**. System call specifying the file handle and where (in memory) the next block of the file should be put. **Read pointer** of the location in the file where the next read is to take place, if sequential.

- **Repositioning withing a file**. *Because a process is usually either reading from or writing to a file, the current operation location can be kept as a per-process **current-file-position pointer**. Both the read and write operations use the same pointer, saving space and reducing system complexity.* The ***current-file-position pointer*** of the open file is positioned to a given value.

- **Deleting a file**. We search the directory for the named file. We release all file space, so that it can be reused by other files, and erase or mark as free the directory entry. Note that some systems allow **hard links**, in this case the actual file content is not deleted until the last link is deleted.

- **Truncating a file**. Erase the contents of a file but keep its atributes.

These primitive operations can be combined to perform other file operations.

Most of the file operations mentioned involve searching the directory for the entry associated with the named file. To avoid this constant searching, many systems require that an `open()` call be made before the file is first used. The operating system keeps a table, called the **open-file table**, containing information about all open files. `create()` and `delete()` are system calls that work with closed rather than open files.


