**Java I/O Masterclass Snippet Set**—a single reference covering everything from basic file handling to advanced streams and NIO. Each snippet is production-ready, with explanations and best practices.

---

## **Java I/O Masterclass**

---

### **1. File & Path Basics**

```java
import java.nio.file.*;

public class FilePathMaster {
    public static void main(String[] args) throws Exception {
        Path dir = Paths.get("data/files");
        if (!Files.exists(dir)) Files.createDirectories(dir);

        Path file = dir.resolve("example.txt");
        if (!Files.exists(file)) Files.createFile(file);

        System.out.println("Directory: " + dir.toAbsolutePath());
        System.out.println("File: " + file.toAbsolutePath());
    }
}
```

✅ Modern approach over `java.io.File`. Use `Files` & `Paths` for flexibility and cross-platform consistency.

---

### **2. Writing to a File**

**Simple Write with `Files.write`:**

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.util.List;

public class WriteFileMaster {
    public static void main(String[] args) throws Exception {
        Path file = Paths.get("data/files/example.txt");
        List<String> lines = List.of("Line 1", "Line 2", "Line 3");
        Files.write(file, lines, StandardCharsets.UTF_8, StandardOpenOption.APPEND);
    }
}
```

**BufferedWriter (efficient for large writes):**

```java
import java.nio.file.*;
import java.io.*;

public class BufferedWriterMaster {
    public static void main(String[] args) throws IOException {
        Path file = Paths.get("data/files/example.txt");
        try (BufferedWriter writer = Files.newBufferedWriter(file, StandardOpenOption.APPEND)) {
            writer.write("BufferedWriter line example");
            writer.newLine();
        }
    }
}
```

---

### **3. Reading from a File**

**Read all lines (small files):**

```java
import java.nio.file.*;
import java.util.List;

public class ReadFileMaster {
    public static void main(String[] args) throws Exception {
        Path file = Paths.get("data/files/example.txt");
        List<String> lines = Files.readAllLines(file);
        lines.forEach(System.out::println);
    }
}
```

**BufferedReader (large files, line-by-line):**

```java
import java.io.*;
import java.nio.file.*;

public class BufferedReaderMaster {
    public static void main(String[] args) throws IOException {
        Path file = Paths.get("data/files/example.txt");
        try (BufferedReader br = Files.newBufferedReader(file)) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        }
    }
}
```

---

### **4. Binary Streams**

**FileInputStream (read bytes):**

```java
import java.io.*;
import java.nio.file.*;

public class FileInputStreamMaster {
    public static void main(String[] args) throws IOException {
        Path file = Paths.get("data/files/example.txt");
        try (InputStream in = Files.newInputStream(file)) {
            byte[] buffer = new byte[1024];
            int bytesRead;
            while ((bytesRead = in.read(buffer)) != -1) {
                System.out.write(buffer, 0, bytesRead);
            }
        }
    }
}
```

**FileOutputStream (write bytes):**

```java
import java.io.*;
import java.nio.file.*;

public class FileOutputStreamMaster {
    public static void main(String[] args) throws IOException {
        Path file = Paths.get("data/files/example-copy.txt");
        try (OutputStream out = Files.newOutputStream(file)) {
            out.write("Writing bytes example".getBytes());
        }
    }
}
```

**Buffered Streams (efficient copy):**

```java
import java.io.*;
import java.nio.file.*;

public class BufferedStreamMaster {
    public static void main(String[] args) throws IOException {
        Path source = Paths.get("data/files/example.txt");
        Path target = Paths.get("data/files/example-copy.txt");

        try (BufferedInputStream bis = new BufferedInputStream(Files.newInputStream(source));
             BufferedOutputStream bos = new BufferedOutputStream(Files.newOutputStream(target))) {

            byte[] buffer = new byte[4096];
            int read;
            while ((read = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, read);
            }
        }
    }
}
```

---

### **5. Random Access Files**

```java
import java.io.*;

public class RandomAccessMaster {
    public static void main(String[] args) throws IOException {
        try (RandomAccessFile raf = new RandomAccessFile("data/files/example.txt", "rw")) {
            raf.seek(0); // move pointer to start
            String firstLine = raf.readLine();
            System.out.println("First line: " + firstLine);

            raf.seek(raf.length()); // append at end
            raf.writeBytes("\nAppended via RandomAccessFile");
        }
    }
}
```

✅ Useful for file updates without reading the entire file into memory.

---

### **6. Object Streams (Serialization)**

```java
import java.io.*;

class Person implements Serializable {
    private String name;
    private int age;
    public Person(String n, int a){ name=n; age=a; }
    public String toString(){ return name + " - " + age; }
}

public class ObjectStreamMaster {
    public static void main(String[] args) throws Exception {
        Person p = new Person("Alice", 30);

        // Write object
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.obj"))) {
            oos.writeObject(p);
        }

        // Read object
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.obj"))) {
            Person readP = (Person) ois.readObject();
            System.out.println(readP);
        }
    }
}
```

---

### **7. NIO Channels & ByteBuffer**

```java
import java.nio.file.*;
import java.nio.channels.*;
import java.nio.*;

public class NIOChannelMaster {
    public static void main(String[] args) throws Exception {
        Path file = Paths.get("data/files/example.txt");
        try (FileChannel channel = FileChannel.open(file, StandardOpenOption.READ)) {
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            while(channel.read(buffer) > 0) {
                buffer.flip();
                while(buffer.hasRemaining()) {
                    System.out.print((char) buffer.get());
                }
                buffer.clear();
            }
        }
    }
}
```

✅ Efficient for large file transfers or memory-mapped I/O.

---

### **8. WatchService (File Change Monitoring)**

```java
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;

public class WatchServiceMaster {
    public static void main(String[] args) throws Exception {
        Path dir = Paths.get("data/files");
        WatchService watchService = FileSystems.getDefault().newWatchService();
        dir.register(watchService, ENTRY_CREATE, ENTRY_DELETE, ENTRY_MODIFY);

        System.out.println("Watching directory for changes...");
        while (true) {
            WatchKey key = watchService.take();
            for (WatchEvent<?> event : key.pollEvents()) {
                System.out.println("Event: " + event.kind() + " on " + event.context());
            }
            key.reset();
        }
    }
}
```

✅ Great for monitoring log directories or dynamic file updates.

---

### **9. Utility Operations**

```java
import java.nio.file.*;

public class FileUtilsMaster {
    public static void main(String[] args) throws Exception {
        Path dir = Paths.get("data/files");

        // List files
        Files.list(dir).forEach(System.out::println);

        // Check file size
        Path file = dir.resolve("example.txt");
        System.out.println("Size: " + Files.size(file) + " bytes");

        // Delete file
        Path tempFile = dir.resolve("example-temp.txt");
        if (Files.exists(tempFile)) Files.delete(tempFile);
    }
}
```

---

### ✅ **Key Takeaways**

- Use **`Paths` + `Files`** for modern I/O.
    
- Use **Buffered streams** for efficiency on large files.
    
- **RandomAccessFile** for partial reads/writes.
    
- **Object streams** for serialization.
    
- **NIO Channels** for high-performance I/O.
    
- **WatchService** for monitoring file system events.
    
- Prefer `try-with-resources` to automatically close streams.
    

---

I can also create a **single GitHub-ready Java project** with all these snippets organized as modules:

- `basic` (Files, Paths)
    
- `text` (Readers/Writers)
    
- `binary` (Streams)
    
- `advanced` (RandomAccess, Serialization, NIO, WatchService)
    

