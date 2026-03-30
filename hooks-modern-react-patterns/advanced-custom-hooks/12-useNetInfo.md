# useNetInfo Hook - Network Connectivity Detection

## 📌 Definition

**useNetInfo** (from `react-native-netinfo`) is a hook that provides information about device network connectivity. It detects when the device goes online/offline, network type (WiFi, cellular, etc.), and connection details.

---

## 🎯 Purpose & What It Does

- **Detects network status** - online or offline
- **Identifies network type** - WiFi, mobile, Bluetooth, etc.
- **Shows connection details** - signal strength, IP addresses
- **Listens to changes** - updates when connectivity changes
- **Perfect for offline handling** - disable features when offline

---

## 💡 Real-World Examples

### Example 1: Show Offline Banner

```javascript
import { useNetInfo } from "@react-native-netinfo";
import { View, Text } from "react-native";

function App() {
  const netInfo = useNetInfo();

  return (
    <View style={{ flex: 1 }}>
      {!netInfo.isConnected && (
        <View style={{ backgroundColor: "red", padding: 10 }}>
          <Text style={{ color: "white" }}>No Internet Connection</Text>
        </View>
      )}

      {/* Rest of app */}
      <Text>You are {netInfo.isConnected ? "online" : "offline"}</Text>
    </View>
  );
}
```

### Example 2: Network Type Detection

```javascript
import { useNetInfo } from "@react-native-netinfo";
import { View, Text } from "react-native";

function ConnectionStatus() {
  const netInfo = useNetInfo();

  const getNetworkTypeLabel = (type) => {
    const labels = {
      wifi: "📶 WiFi",
      cellular: "📱 Cellular",
      bluetooth: "🎧 Bluetooth",
      ethernet: "🖥️ Ethernet",
      unknown: "❓ Unknown",
      none: "❌ None",
    };
    return labels[type] || "Unknown";
  };

  return (
    <View>
      <Text>Connected: {netInfo.isConnected ? "Yes" : "No"}</Text>
      <Text>Type: {getNetworkTypeLabel(netInfo.type)}</Text>
      {netInfo.isConnected && (
        <>
          <Text>
            Details:{" "}
            {netInfo.details?.isConnectionExpensive ? "Expensive" : "Normal"}
          </Text>
        </>
      )}
    </View>
  );
}
```

### Example 3: Disable Heavy Features When on Cellular

```javascript
import { useNetInfo } from "@react-native-netinfo";
import { View, TouchableOpacity, Text } from "react-native";

function VideoStreaming() {
  const netInfo = useNetInfo();

  const isExpensiveConnection =
    netInfo.type === "cellular" && netInfo.details?.isConnectionExpensive;

  const handlePlayVideo = () => {
    if (isExpensiveConnection) {
      Alert.alert(
        "Expensive Connection",
        "Video streaming on cellular may use a lot of data. Continue?",
        [{ text: "Cancel" }, { text: "Play", onPress: () => startStreaming() }],
      );
    } else {
      startStreaming();
    }
  };

  return (
    <TouchableOpacity onPress={handlePlayVideo}>
      <Text>
        Play Video
        {isExpensiveConnection && " ⚠️ (Using cellular)"}
      </Text>
    </TouchableOpacity>
  );
}
```

### Example 4: Queue Operations When Offline

```javascript
import { useNetInfo } from "@react-native-netinfo";
import { useState } from "react";
import { View, TouchableOpacity, Text, FlatList } from "react-native";

function OfflineQueue() {
  const netInfo = useNetInfo();
  const [queue, setQueue] = useState([]);

  const handleSubmit = (data) => {
    if (!netInfo.isConnected) {
      // Queue for later
      setQueue([...queue, { id: Date.now(), data, timestamp: new Date() }]);
      Alert.alert("Queued", "Will sync when online");
    } else {
      // Send immediately
      uploadData(data);
    }
  };

  return (
    <View>
      <Text>Queue: {queue.length} items</Text>
      {queue.length > 0 && (
        <FlatList
          data={queue}
          renderItem={({ item }) => (
            <View style={{ padding: 10, borderBottomWidth: 1 }}>
              <Text>Pending since {item.timestamp.toLocaleTimeString()}</Text>
            </View>
          )}
        />
      )}
      <TouchableOpacity onPress={() => handleSubmit({ data: "test" })}>
        <Text>Submit</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### Example 5: Network Status Monitor

```javascript
import { useNetInfo } from "@react-native-netinfo";
import { useState, useEffect } from "react";
import { View, Text, ScrollView } from "react-native";

function NetworkMonitor() {
  const netInfo = useNetInfo();
  const [isConnected, setIsConnected] = useState(null);

  useEffect(() => {
    if (netInfo.isConnected !== isConnected && isConnected !== null) {
      const message = netInfo.isConnected
        ? "🟢 Connected to the internet"
        : "🔴 Disconnected from the internet";
      console.log(message);
    }
    setIsConnected(netInfo.isConnected);
  }, [netInfo.isConnected]);

  return (
    <ScrollView style={{ padding: 20 }}>
      <Text style={{ fontSize: 18, fontWeight: "bold", marginBottom: 10 }}>
        Network Status
      </Text>
      <Text>Connected: {netInfo.isConnected ? "Yes ✓" : "No ✗"}</Text>
      <Text>Type: {netInfo.type}</Text>
      <Text>Expensive: {netInfo.isInternetReachable ? "N/A" : "Unknown"}</Text>
      {netInfo.details && (
        <>
          <Text>SSID: {netInfo.details.ssid || "N/A"}</Text>
          <Text>IP: {netInfo.details.ipAddress || "N/A"}</Text>
        </>
      )}
    </ScrollView>
  );
}
```

---

## ❓ Interview Q&A

**Q1: What information does useNetInfo provide?**

A: isConnected, type (wifi/cellular/etc), isInternetReachable, details (varies by platform).

```javascript
const netInfo = useNetInfo();
// {
//   isConnected: true,
//   isInternetReachable: true,
//   type: 'wifi',
//   details: {
//     ssid: 'MyNetwork',
//     ipAddress: '192.168.1.1',
//     ...
//   }
// }
```

**Q2: What are the different network types?**

A: wifi, cellular, bluetooth, ethernet, unknown, none

```javascript
switch (netInfo.type) {
  case "wifi":
    return "WiFi connected";
  case "cellular":
    return "Mobile data";
  case "bluetooth":
    return "Bluetooth";
  default:
    return "Other";
}
```

**Q3: How do you detect expensive connections?**

A: Check if type is 'cellular' or use isConnectionExpensive flag.

```javascript
const isExpensive =
  netInfo.type === "cellular" || netInfo.details?.isConnectionExpensive;
```

**Q4: What's the difference between isConnected and isInternetReachable?**

A:

- `isConnected` - Device has network connection (local)
- `isInternetReachable` - Device can reach the internet

```javascript
// Device might be on WiFi but WiFi has no internet
netInfo.isConnected; // true
netInfo.isInternetReachable; // false
```

---

## ⚠️ Common Mistakes

### ❌ Mistake 1: Using Network Status Without Checking Connectivity

```javascript
// BAD - might send data when offline
async function uploadData(data) {
  const response = await fetch("/api/upload", { body: data });
}

// GOOD - check connection first
const netInfo = useNetInfo();

async function uploadData(data) {
  if (!netInfo.isConnected) {
    saveForLater(data);
    return;
  }
  const response = await fetch("/api/upload", { body: data });
}
```

### ❌ Mistake 2: Not Handling Expensive Networks

```javascript
// BAD - auto-download on cellular
useEffect(() => {
  downloadLargeFile();
}, []);

// GOOD - warn user first
useEffect(() => {
  if (netInfo.type === "cellular") {
    askUserConfirmation();
  } else {
    downloadLargeFile();
  }
}, [netInfo.type]);
```

---

## 📝 Practice Exercise

**Task:** Create a sync manager that:

1. Shows network status
2. Queues operations when offline
3. Auto-syncs when coming online
4. Shows sync progress

**Solution:**

```javascript
import { useNetInfo } from "@react-native-netinfo";
import { useState, useEffect } from "react";
import { View, Text, TouchableOpacity, FlatList, Alert } from "react-native";

function SyncManager() {
  const netInfo = useNetInfo();
  const [queue, setQueue] = useState([]);
  const [isSyncing, setIsSyncing] = useState(false);

  // Auto-sync when coming online
  useEffect(() => {
    if (netInfo.isConnected && queue.length > 0) {
      syncQueue();
    }
  }, [netInfo.isConnected]);

  const addToQueue = (data) => {
    setQueue([...queue, { id: Date.now(), data, status: "pending" }]);
  };

  const syncQueue = async () => {
    if (isSyncing) return;
    setIsSyncing(true);

    for (const item of queue) {
      try {
        const response = await fetch("/api/sync", {
          method: "POST",
          body: JSON.stringify(item.data),
        });
        if (response.ok) {
          setQueue((q) => q.filter((x) => x.id !== item.id));
        }
      } catch (error) {
        console.error("Sync failed:", error);
      }
    }

    setIsSyncing(false);
    if (queue.length === 0) {
      Alert.alert("Sync Complete", "All items synced!");
    }
  };

  return (
    <View style={{ flex: 1, padding: 20 }}>
      {/* Network Status */}
      <View
        style={{
          padding: 10,
          backgroundColor: netInfo.isConnected ? "#e8f5e9" : "#ffebee",
          marginBottom: 20,
          borderRadius: 5,
        }}
      >
        <Text style={{ fontWeight: "bold" }}>
          {netInfo.isConnected ? "🟢 Connected" : "🔴 Offline"}
        </Text>
        <Text>Type: {netInfo.type}</Text>
      </View>

      {/* Queue Status */}
      <Text style={{ fontSize: 16, fontWeight: "bold", marginBottom: 10 }}>
        Pending Items: {queue.length}
      </Text>

      {/* Queue List */}
      <FlatList
        data={queue}
        renderItem={({ item }) => (
          <View style={{ padding: 10, borderBottomWidth: 1 }}>
            <Text>{item.data.description || JSON.stringify(item.data)}</Text>
            <Text style={{ fontSize: 12, color: "gray" }}>
              {item.status === "syncing" ? "⏳ Syncing..." : "⏸️ Pending"}
            </Text>
          </View>
        )}
        keyExtractor={(item) => item.id.toString()}
      />

      {/* Manual Add & Sync */}
      <TouchableOpacity
        onPress={() =>
          addToQueue({ description: "Test item", timestamp: new Date() })
        }
        style={{ padding: 10, backgroundColor: "blue", marginTop: 10 }}
      >
        <Text style={{ color: "white" }}>Add Item</Text>
      </TouchableOpacity>

      <TouchableOpacity
        onPress={syncQueue}
        disabled={!netInfo.isConnected || isSyncing || queue.length === 0}
        style={{ padding: 10, backgroundColor: "green", marginTop: 10 }}
      >
        <Text style={{ color: "white" }}>
          {isSyncing ? "Syncing..." : "Sync Now"}
        </Text>
      </TouchableOpacity>
    </View>
  );
}

export default SyncManager;
```

---

## 🎓 Key Takeaways

✅ **Detects network connectivity** - online/offline status  
✅ **Identifies network type** - WiFi, cellular, Bluetooth  
✅ **Handles expensive networks** - warn on cellular  
✅ **Enables offline features** - queue operations  
✅ **Perfect for sync features** - auto-sync when online

---

**Next:** Learn about useAppState for app lifecycle management.
