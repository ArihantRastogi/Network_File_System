![NFS Image](Working-of-DFS.webp)
Credit : GeeksForGeeks


<h1 align="center">Network File System (NFS)</h1>

## Project Background

This project was developed as part of the _CS3.301 Operating Systems and Networks at IIIT-Hyderabad_.

The primary goal was to apply the skills and knowledge acquired throughout the course to create a functional Network File System (NFS). This hands-on experience has significantly deepened our understanding of networking concepts and enhanced our ability to collaborate on larger projects within a team setting.

## Introduction

The Network File System (NFS) project is designed to facilitate efficient file management and access across a network. This system includes several key components that work together to provide a seamless experience for clients interacting with files stored on various servers:

- **Clients**: Clients represent the systems or users requesting access to files within the NFS. They serve as the primary interface to initiate various file-related operations such as reading, writing, deleting, and streaming.
  
- **Naming Server (NM)**: The NM acts as a central hub orchestrating communication between clients and storage servers. Its primary function is to provide clients with crucial information about the specific storage server where a requested file or folder resides, essentially acting as a directory service.

- **Storage Servers (SS)**: These servers are responsible for the physical storage and retrieval of files and folders, managing data persistence and distribution across the network.

## User Functionalities

Clients enjoy a suite of essential file operations, enabling seamless interaction with the NFS:

- **Writing a File/Folder**: Clients can create and update files and folders within the NFS, including synchronous and asynchronous writes for optimized response times.

- **Reading a File**: Clients can retrieve contents of files stored within the NFS.

- **Deleting a File/Folder**: Clients can remove files and folders from the NFS when they are no longer needed.

- **Creating a File/Folder**: Clients can generate new files and folders, facilitating expansion and organization of the file system.

- **Listing All Files and Folders in a Folder**: Clients can retrieve comprehensive listings of files and subfolders within specified directories.

- **Getting Additional Information**: Clients can access metadata about specific files, including size, access rights, timestamps, etc.

- **Streaming Audio Files**: Clients can stream audio files directly over the NFS.

- **Listing All Accessible Paths**: Clients can request the NM to provide all accessible paths available across registered storage servers. This helps clients understand what files they can access.

## Features 

### 1. Initialization of Storage and Naming Servers

- **Initialize Naming Server (NM)**: Naming Server uses dynamic IP and can be initialized on any machine. 
- **Initialize Storage Server 1 (SS_1)**: Storage Server using the IP and Port of the Naming server, connects to the naming server and upon initialization, SS_1 sends vital details about its existence to NM:
     - IP address
     - Port for NM Connection
     - Port for Client Connection
     - List of Accessible Paths
    Similarly other storage server can be initialized.
- **Dynamic Addition of Storage Servers**: 
    New SS can register with NM at any time during execution. This allows our system to scale online.
  
### 2. Naming Server acts as a center orchestrator : 

- **Storing Storage Server Data**: Naming Server acts as a central repository for information provided by SS.

- **Handling Client Requests** : 
    #### KIND 1 Requests : 
    - **Setting up connection between Client and the Storage Server**: For these request the Naming Server returns the IP and Port for Client Interaction for the Storage Server over which the requested file has been stored.

        - Read a File
        - Write to a File
        - Get Size and Permissions
        - Stream Audio Files

    #### KIND 2 Requests : 
    - **Handling Request on Behalf of Clients**: These requests are handled by the Naming Server directly.

        - Create an Empty File/Directory
        - Delete a File/Directory
        - Copy Files/Directories from other SS

    #### KIND 3 Requests : 
    - **Listing All Accessible Paths** : The Naming Server prints all the paths that the clients can access. 

- **Client Task Feedback (Acknowledging Clients)**: Provides timely feedback to clients upon task completion, ensuring efficient communication within the system.



### 3. Advanced Functionalities

- **Multiple Clients** : 
    - The NFS design accommodates multiple clients accessing the NM simultaneously without blocking operations for any particular client.
    - *Concurrent File Reading*: Multiple clients can read from the same file simultaneously; however, only one client can write at any given time due to potential conflicts during asynchronous writes.

- **Error Codes**: A set of distinct error codes defined for situations where requests cannot be fulfilled—such as when files do not exist or are currently being written by another client—enhancing communication regarding issues between NFS and clients.

- **Efficient Implementation of Naming Server**:
   - Optimized ther search processes used by NM by employing efficient data structures Tries for faster identification of appropriate Storage Servers for requests.
   - *LRU Caching*: Implemented LRU (Least Recently Used) caching for recent searches to expedite subsequent requests for frequently accessed data, improving response times significantly.


- **Bookkeeping** : Implemented logging mechanisms where NM records every request or acknowledgment received from clients or Storage Servers along with relevant IP addresses and ports used in communications for traceability and debugging purposes.

- **Backing Up Data**: 
    - *Detection* : Naming Server uses a special *Ping* message before sending it some data to check if a Storage Server is live or not.
    - *Replication* : Every file/folder stored within an SS across two other SS once more than two exist; this ensures data availability even during failures while allowing only read operations until recovery is complete.

- **Redundancy**:  When an SS reconnects after downtime, we ensure that duplicated storage servers are matched back with original ones without adding new entries during this process.

- **Allowing Asynchronous Writing**: 
    - The NFS supports asynchronous writes for improved performance, especially with large data.
    - Clients receive immediate acknowledgment upon request, allowing them to proceed with other tasks.
    - The Storage Server (SS) writes data in chunks to persistent memory.
    - After successful completion, the SS informs the client through the Naming Server (NM).


### Installation

To set up the NFS project locally:

1. Clone this repository to your local machine.
   
2. Ensure you have all necessary dependencies installed.

3. Configure IP addresses for your Naming Server and Storage Servers as required.

4. Start the Naming Server followed by the Storage Servers.

5. Finally, launch client applications to interact with the NFS.
---


## Assumptions

### Storage Server
- While the storage server is down, we are assuming no modifications such as adding / deleting / updating new files happen on the Storage server. The state remains exactly the same. 
- The UNIQUE ID assigned to the SS does not change. (In real world the ID can be thought of something unique to a physical storage server like its MAC Address). 
- The SS_PORT is unique for each storage server. It is defined as a macro and thus needs to be manually changed when creating and running a new storage server. 
- The S_SSPORT is for ss-ss comunications and is common for all storage servers.
- The storage servers need to have different IPs to differentiate them on same network and hence no two storage servers can run on same machine with same IP.
- The naming server IP and Port is made known to each storage server by using CLI.

### Naming Server
- The naming server has two pre-defined ports for back-communication for handling ASYNC write acknowledgements. 
- During Write request, we assume that no delete or copy requests from any client.
- When any storage server comes back online, only the unique ID is matched in the ss_serverlist to check if the storage server had previously been connected.

### Client Server
- The naming server IP and Port is made known to each client by using CLI.
- Each storage server upon starting has access to it's own database in the form of a folder `SS_<UNIQUE_ID>`. One storage server cannot access other storage server's private folder.
- A file and a folder with the same name cannot exist.

## Important Macros 
- In `Naming Server`
    -   ```
        // Error Codes
        #define SUCCESS 0
        #define FILE_ALREADY_EXISTS 400
        #define FILE_NOT_FOUND 404
        #define STORAGE_SERVER_NOT_FOUND 505
        #define ACCESS_DENIED 999
        #define FILE_NOT_CREATED 1001
        #define FILE_NOT_DELETED 1000
        #define INVALID_PATH 100
        #define CLIENT_SERVER_DISCONNECTED 200
        #define TIMEOUT 600
        ```
    -   ```
        #define MAXCLIENTS 100
        #define MAXPATHLEN 200
        #define MAX_FILES 100
        #define MAXFILENAME 100
        #define NS_PORT 8080       // Port for connecting clients
        #define NS_PORT_SS 8000    // Port for connecting storage servers
        #define MAXSSERVER 100
        #define ALPHABET_SIZE 256
        #define MAXDATASIZE 100
        #define CACHE_SIZE 2
        ```
    -   ```
        #define BACK_CHANNEL_PORT 5113 
        ```
- In `SS`
    -   ```
        #define SS_PORT 8005  // Port for client connections
        #define S_SSPORT 8010 // for ss-ss comms
        #define BACKUP_PORT 8011
        #define MAX_BUFFER 1024
        #define MAXPATHLEN 200
        #define MAXFILENAME 100
        #define MAX_FILES 100
        #define UNIQUE_ID 100
        #define BUFFER_SIZE 1024
        #define MAXSSERVER 100
        #define MAXDATASIZE 100
        #define WRITEBUFFER 4096
        ```
- In `Client`
    -   ```
        # define MAXDATASIZE 100
        # define MAXPATHLEN 200
        # define MAXFILENAME 100
        #define CMD_READ 1 // Kind 1 request
        #define CMD_WRITE 2 // Kind 1 request 
        #define CMD_DELETE 3 // Kind 2 request
        #define CMD_GET_INFO 4 // Kind 1 request
        #define CMD_STREAM 5 // Kind 
        #define CMD_CREATE 6 // Kind 
        #define CMD_COPY 7 
        #define CMD_LIST 8
        #define CMD_APPEND 9
        #define CMD_WRITE_ASYNC 10

        # define EXIT_STATUS 10 
        # define CLEAR_STATUS 9
        # define KIND_1 1 
        # define KIND_2 2 
        # define KIND_3 3
        ```
## User View
### Running the NFS
- First run name server by going into `Naming server` directory by using `cd` command.
```
Naming server % make
gcc ns.c LRU.c backcommunication.c logging.c clienthandler.c sserverhandler.c trie.c locks.c -lpthread -o ns.o
Naming server % ./ns.o
```
- Then connect all storage server (Ensuring unique values of UNIQUE_ID and SS_PORT) by running the following steps in `SS` folder.
```
SS % make
gcc ss.c func.c read_write.c namingserverhandler.c clientserverhandler.c storageserverhandler.c threadhandler.c copy.c -o ss.o -pthread -lm
SS % ./ss.o <naming_server_ip> <naming_server_port_for_ss>
```
- The `<naming_server_port_for_ss>` is given by `NS_PORT_SS` in `headers.h` of `Naming server`.
- Then the clients can connect by running the following steps in `Client` folder.
```
Client % make
gcc client.c -o client 
Client % ./client <naming_server_ip> <naming_server_port_for_client>
```
- The `<naming_server_port_for_client>` is given by `NS_PORT` in `headers.h` of `SS`.

### Commands
- As soon as the client logs in, it is greeted with a help message that prints all commands available to the user
```
Commands available:
  READ <path> - Read a file
  WRITE <path> - Write to a file
  WRITEASYNC <path> - Write to a file asynchronously
  DELETE <directory_path> <filename>- Delete a file
  DELETE <directory> - Delete a directory
  GETINFO <path> - Get information about a file or directory
  STREAM <path> - Stream a music file
  CREATE <path> <name> - Create a file
  CREATE <directory_path> <name> - Create a folder
  COPY <source> <dest> - Copy a file
  APPEND <path> - Append to a file
  LIST <path> - List all files in a directory (Press ~ for root)
  HELP - Show this help message
  clear - Clear the screen
  exit - Exit the program
Connected successfully to port : 8000
```
- Here is the list of all the commands the user can use to interact with the NFS.
    - `READ <path>`: Reads the content of the specified file from the storage server. The path is from the root directory however it is not needed to be specified.
        - USAGE
            - For the below file structure. 
            ```
            file.txt // in root directory
            abc/file.txt
            dir1/dir2/file.txt
            ```
            - The user can access the text files by running below commands.
            ```
            READ file.txt
            READ abc/file.txt
            READ dir1/dir2/file.txt
            ```
    - `WRITE <path>`: Writes data to the specified file on the storage server.
        - USAGE
            - For the below file structure.
            ```
            file.txt // in root directory
            abc/file.txt
            dir1/dir2/file.txt
            ```
            - The user can write to the text files by running below commands.
            ```
            WRITE file.txt
            WRITE abc/file.txt
            WRITE dir1/dir2/file.txt
            ```
            - For writing the content
            ```
            > > hi this is line 1
            > this is line 2
            > END
            ```
            - `END\n` signifies the end of the input.
    - `WRITEASYNC <path>`: Writes data to the specified file asynchronously on the storage server.
        - USAGE
            - For the below file structure.
            ```
            file.txt // in root directory
            abc/file.txt
            dir1/dir2/file.txt
            ```
            - The user can write asynchronously to the text files by running below commands.
            ```
            WRITEASYNC file.txt
            WRITEASYNC abc/file.txt
            WRITEASYNC dir1/dir2/file.txt
            ```
        - ACKS
            - After a specified timeout or if the content is written to file, an ACK is received by the client along with respective error codes to ensure whether the WRITE was succesful or not. 
    - `DELETE <directory_path> <filename>`: Deletes the specified file from the directory on the naming server.
        - USAGE
            - For the below file structure.
            ```
            file.txt
            abc/file.txt
            dir1/dir2/file.txt
            ```
            - The user can delete the text files by running below commands.
            ```
            DELETE . file.txt
            DELETE abc file.txt
            DELETE dir1/dir2 file.txt
            ```
    - `DELETE <directory>`: Deletes the specified directory from the naming server.
        - USAGE
            - For the below directory structure.
            ```
            abc/
            dir1/dir2/
            ```
            - The user can delete the directories by running below commands.
            ```
            DELETE abc
            DELETE dir1/dir2
            ```
        - `DELETE .` is a restricted command and user cannot delete the rrot directory by using this command. However they can clear the root directory by indivsuallly removing all file paths and directories in the root directory.
    - `GETINFO <path>`: Retrieves information about the specified file or directory from the storage server.
        - USAGE
            - For the below file structure.
            ```
            file.txt // in root directory
            abc/file.txt
            dir1/dir2/file.txt
            ```
            - The user can get information about the text files by running below commands.
            ```
            GETINFO file.txt
            GETINFO abc/file.txt
            GETINFO dir1/dir2/file.txt
            ```
        - OUTPUT
            ```
            File Information:
            Size: 18 bytes
            Permissions: -rw-r--r--
            Owner: dheeru
            Group: staff
            Last Modified: Nov 20 03:03
            Number of Links: 1
            ```
    - `STREAM <path>`: Streams the specified music file from the storage server.
        - USAGE
            - For the below file structure.
            ```
            song.mp3 // in root directory
            abc/song.mp3
            dir1/dir2/song.mp3
            ```
            - The user can stream the music files by running below commands.
            ```
            STREAM song.mp3
            STREAM abc/song.mp3
            STREAM dir1/dir2/song.mp3
            ```
            - The mpv player spawn a new terminal with the audio/video player with default controls. Closing it returns normal function for the user.
    - `CREATE <path> <name>`: Creates a new file with the specified name at the given path on the naming server.
        - USAGE
            - For the below directory structure.
            ```
            abc/
            dir1/dir2/
            ```
            - The user can create new text files by running below commands.
            ```
            CREATE . newfile.txt // for creating in root directory
            CREATE abc newfile.txt
            CREATE dir1/dir2 newfile.txt
            ```
    - `CREATE <directory_path> <name>`: Creates a new folder with the specified name at the given directory path on the naming server.
        - USAGE
            - For the below directory structure.
            ```
            abc/
            dir1/dir2/
            ```
            - The user can create new directories by running below commands.
            ```
            CREATE . xyz // for creating in root directory
            CREATE abc newdir
            CREATE dir1/dir2 newdir
            ```
    - `COPY <source> <dest>`: Copies the specified file from the source path to the destination path on the naming server.
        - USAGE
            - For the below file structure.
            ```
            file.txt // in root directory
            abc/file2.txt
            dir1/dir2/file3.txt
            ```
            - The user can copy the text files or directories to a destination directory by running below commands.
            ```
            COPY file.txt abc
            COPY abc/file2.txt dir1/dir2
            COPY dir1/dir2/file3.txt . // for copying to root directory
            COPY . abc // for copying root directory to specified directory
            ```
            - The user can copy only to directories. The code will raise an exception if the `dest` is not a directory. 
    - `APPEND <path>`: Appends data to the specified file on the storage server.
        - USAGE
            - Exactly the same as WRITE
    - `LIST <path>`: Lists all files in the specified directory. Use `~` for the root directory.
        - USAGE
            - For the below directory structure.
            ```
            abc/
            dir1/dir2/
            ```
            - The user can list all files in the directories by running below commands.
            ```
            LIST abc
            LIST dir1/dir2
            LIST ~
            ```
    - `HELP`: Displays the help message with all available commands.
    - `clear`: Clears the screen.
    - `exit`: Exits the program.


## Team Members

- Arihant Rastogi (2023111012)
- Manas Mittal (2024121003)
- Prasson Dev (2023111014)
- Sudheera YS (2023111002)
---