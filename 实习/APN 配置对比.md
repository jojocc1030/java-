要使用Java读取这样格式的.cfg文件，并根据`#`标记的行来识别新的APN条目，并将每个APN的值存储在HashMap中，你可以采用以下步骤和代码实现：

1. 读取文件。
2. 使用逻辑判断`#`行以确定新的APN开始。
3. 将每个APN及其相关配置存储在HashMap中。

下面是一个基本的示例代码，展示如何实现这个过程：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class ConfigReader {
    public static void main(String[] args) {
        String path = "path/to/your/config.cfg"; // 替换为你的文件路径
        Map<String, Map<String, String>> apns = readConfigFile(path);
        System.out.println(apns);
    }

    public static Map<String, Map<String, String>> readConfigFile(String filePath) {
        Map<String, Map<String, String>> apns = new HashMap<>();
        Map<String, String> currentApnConfig = null;
        String currentApn = null;

        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                line = line.trim();
                if (line.startsWith("#")) {
                    // 当遇到 # 时，认为是一个新的APN开始
                    if (currentApn != null) {
                        apns.put(currentApn, currentApnConfig);
                    }
                    currentApnConfig = new HashMap<>();
                    currentApn = null;
                } else if (line.startsWith("apn")) {
                    // 读取apn行并设置当前apn
                    currentApn = line.split("\\s+")[1];
                } else if (!line.isEmpty() && currentApn != null) {
                    // 处理其他配置行
                    String[] parts = line.split("\\s+");
                    if (parts.length > 1) {
                        currentApnConfig.put(parts[0], parts[1]);
                    }
                }
            }
            // 将最后一个APN配置添加到列表
            if (currentApn != null) {
                apns.put(currentApn, currentApnConfig);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return apns;
    }
}
```

在这个例子中，我们创建了一个名为`ConfigReader`的类，它具有读取配置文件并将其存储到由HashMap组成的HashMap中的功能。每个APN的配置都是另一个HashMap，键是配置项，值是配置的值。

你需要将`path/to/your/config.cfg`替换为你的实际.cfg文件路径。这段代码还假设配置文件的格式是标准的，并且每个配置项都遵循`key value`格式。如果你的文件格式稍有不同，可能需要调整代码中的解析逻辑。

了解到你需要读取的.cfg文件中不仅包含APN配置，还包含了以`access view`为起始标志的L2TP配置。这需要对之前的解析逻辑进行一些调整，以便同时处理这两种不同的配置区段。下面是一个更新后的Java代码，它可以同时解析APN和L2TP配置。

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class ConfigReader {
    public static void main(String[] args) {
        String path = "path/to/your/config.cfg"; // 替换为你的文件路径
        Map<String, Object> configurations = readConfigFile(path);
        System.out.println(configurations);
    }

    public static Map<String, Object> readConfigFile(String filePath) {
        Map<String, Object> configurations = new HashMap<>();
        Map<String, String> currentApnConfig = null;
        Map<String, String> currentL2tpConfig = null;
        String currentApn = null;
        boolean inL2tpSection = false;

        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                line = line.trim();
                if (line.startsWith("#") && !inL2tpSection) {
                    // 当遇到 # 时，认为是一个新的APN开始
                    if (currentApn != null) {
                        configurations.put(currentApn, currentApnConfig);
                    }
                    currentApnConfig = new HashMap<>();
                    currentApn = null;
                } else if (line.startsWith("access view")) {
                    // 处理L2TP部分的开始
                    if (currentApn != null) {
                        configurations.put(currentApn, currentApnConfig);
                        currentApn = null;
                    }
                    currentL2tpConfig = new HashMap<>();
                    configurations.put("L2TP", currentL2tpConfig);
                    inL2tpSection = true;
                } else if (line.startsWith("apn") && !inL2tpSection) {
                    // 读取apn行并设置当前apn
                    currentApn = line.split("\\s+")[1];
                } else if (!line.isEmpty()) {
                    // 处理其他配置行
                    String[] parts = line.split("\\s+");
                    if (parts.length > 1) {
                        if (inL2tpSection) {
                            currentL2tpConfig.put(parts[0], parts[1]);
                        } else if (currentApn != null) {
                            currentApnConfig.put(parts[0], parts[1]);
                        }
                    }
                }
            }
            // 将最后一个APN或L2TP配置添加到列表
            if (currentApn != null) {
                configurations.put(currentApn, currentApnConfig);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return configurations;
    }
}
```

这段代码通过使用一个标志`inL2tpSection`来跟踪当前是否处于L2TP配置部分。当遇到`access view`时，它将设置此标志，并开始读取L2TP的配置。一旦新的APN开始或文件结束，它会将当前APN或L2TP配置保存到`configurations`映射中。

这种方式允许你将不同类型的配置分开处理，并存储在同一个数据结构中。注意，L2TP配置被存储在键"L2TP"下。如果配置文件中有多个L2TP部分，你可能需要调整代码以支持这种情况，比如使用一个列表来存储每个L2TP配置，而不是一个单独的映射。
如果.cfg文件中有多个L2TP配置部分，我们需要将这些配置存储在一个列表中，而不是单个映射。下面是对上面代码的调整，使其能够处理多个L2TP配置部分：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ConfigReader {
    public static void main(String[] args) {
        String path = "path/to/your/config.cfg"; // 替换为你的文件路径
        Map<String, Object> configurations = readConfigFile(path);
        System.out.println(configurations);
    }

    public static Map<String, Object> readConfigFile(String filePath) {
        Map<String, Object> configurations = new HashMap<>();
        Map<String, String> currentApnConfig = null;
        List<Map<String, String>> l2tpConfigurations = new ArrayList<>();
        String currentApn = null;
        boolean inL2tpSection = false;

        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                line = line.trim();
                if (line.startsWith("#") && !inL2tpSection) {
                    // 当遇到 # 时，认为是一个新的APN开始
                    if (currentApn != null) {
                        configurations.put(currentApn, currentApnConfig);
                    }
                    currentApnConfig = new HashMap<>();
                    currentApn = null;
                } else if (line.startsWith("access view")) {
                    // 处理L2TP部分的开始
                    if (currentApn != null) {
                        configurations.put(currentApn, currentApnConfig);
                        currentApn = null;
                    }
                    if (inL2tpSection) {
                        // 如果已经处于L2TP部分，将当前配置保存并开始新的配置
                        l2tpConfigurations.add(new HashMap<>());
                    } else {
                        // 第一次进入L2TP配置部分
                        l2tpConfigurations.add(new HashMap<>());
                        configurations.put("L2TP", l2tpConfigurations);
                        inL2tpSection = true;
                    }
                } else if (line.startsWith("apn") && !inL2tpSection) {
                    // 读取apn行并设置当前apn
                    currentApn = line.split("\\s+")[1];
                } else if (!line.isEmpty()) {
                    // 处理其他配置行
                    String[] parts = line.split("\\s+");
                    if (parts.length > 1) {
                        if (inL2tpSection) {
                            Map<String, String> currentL2tpConfig = l2tpConfigurations.get(l2tpConfigurations.size() - 1);
                            currentL2tpConfig.put(parts[0], parts[1]);
                        } else if (currentApn != null) {
                            currentApnConfig.put(parts[0], parts[1]);
                        }
                    }
                }
            }
            // 将最后一个APN或L2TP配置添加到列表
            if (currentApn != null) {
                configurations.put(currentApn, currentApnConfig);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return configurations;
    }
}
```

这段代码通过创建一个`ArrayList`来存储所有L2TP配置，每次遇到新的`access view`时，都会向此列表中添加一个新的HashMap。这样，每个L2TP配置都被独立保存，且可以处理多个L2TP配置区块。此外，我们在`configurations`中用"L2TP"作为键，将整个列表作为值存储。

这种方式确保了代码可以灵活地处理含有多个L2TP配置段的情况。