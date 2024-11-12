# Smart-EV-Charging-Algorithm

    import java.util.*;
	public class SmartEVChargingSystem {
     private static final int MAX_SYSTEM_CAPACITY = 92;
     private static final int MAX_DEVICE_CAPACITY = 40;
   
	//Device class to store device information
    static class Device {
        String id;
        int connectionTime;
        int currentConsumption;

        Device(String id, int connectionTime, int currentConsumption) {
            this.id = id;
            this.connectionTime = connectionTime;
            this.currentConsumption = currentConsumption;
        }
    }

    // Main charging management class
    static class ChargingManager {
        // Queue to maintain FIFO order of devices
        private Queue<Device> deviceQueue;
        
        // Map to track current device consumption
        private Map<String, Device> activeDevices;
        
        // Current total system consumption
        private int totalConsumption;

        public ChargingManager() {
            deviceQueue = new LinkedList<>();
            activeDevices = new HashMap<>();
            totalConsumption = 0;
        }

        // Connect a new device
        public void connectDevice(String deviceId, int timestamp) {
            // Determine initial consumption (max or remaining capacity)
            int initialConsumption = calculateInitialConsumption();
            
            Device newDevice = new Device(deviceId, timestamp, initialConsumption);
            
            deviceQueue.offer(newDevice);
            activeDevices.put(deviceId, newDevice);
            
            // Update total consumption
            totalConsumption += initialConsumption;
            
            // Rebalance if necessary
            rebalanceConsumption();
        }

        // Disconnect a device
        public void disconnectDevice(String deviceId) {
            Device deviceToRemove = activeDevices.get(deviceId);
            
            if (deviceToRemove != null) {
                // Reduce total consumption
                totalConsumption -= deviceToRemove.currentConsumption;
                
                // Remove from queue and active devices
                deviceQueue.removeIf(d -> d.id.equals(deviceId));
                activeDevices.remove(deviceId);
                
                // Rebalance remaining devices
                rebalanceConsumption();
            }
        }

        // Change device consumption
        public void changeDeviceConsumption(String deviceId, int newConsumption) {
            Device device = activeDevices.get(deviceId);
            
            if (device != null) {
                // Adjust total consumption
                totalConsumption -= device.currentConsumption;
                device.currentConsumption = newConsumption;
                totalConsumption += newConsumption;
                
                // Rebalance if necessary
                rebalanceConsumption();
            }
        }

        // Calculate initial consumption for new device
        private int calculateInitialConsumption() {
            int remainingCapacity = MAX_SYSTEM_CAPACITY - totalConsumption;
            return Math.min(MAX_DEVICE_CAPACITY, remainingCapacity);
        }

        // Rebalance power consumption across devices
        private void rebalanceConsumption() {
            // If total consumption exceeds system capacity
            if (totalConsumption > MAX_SYSTEM_CAPACITY) {
                // Sort devices by connection time (FIFO)
                List<Device> sortedDevices = new ArrayList<>(deviceQueue);
                sortedDevices.sort(Comparator.comparingInt(d -> d.connectionTime));
                
                // Reduce consumption from earliest connected devices
                for (Device device : sortedDevices) {
                    if (totalConsumption <= MAX_SYSTEM_CAPACITY) {
                        break;
                    }
                    
                    // Reduce consumption
                    int reductionAmount = Math.min(
                        device.currentConsumption, 
                        totalConsumption - MAX_SYSTEM_CAPACITY
                    );
                    
                    device.currentConsumption -= reductionAmount;
                    totalConsumption -= reductionAmount;
                }
            }
        }

        // Print current device consumptions
        public void printDeviceConsumptions() {
            System.out.println("Current Device Consumptions:");
            for (Device device : activeDevices.values()) {
                System.out.println(device.id + ": " + device.currentConsumption + " units");
            }
            System.out.println("Total Consumption: " + totalConsumption + " units");
        }
    }

    // Demonstration
    public static void main(String[] args) {
        ChargingManager chargingManager = new ChargingManager();

        // Scenario demonstration
        chargingManager.connectDevice("Device A", 0);  // 40 units
        chargingManager.printDeviceConsumptions();

        chargingManager.connectDevice("Device B", 1);  // 40 units
        chargingManager.printDeviceConsumptions();

        chargingManager.connectDevice("Device C", 2);  // 12 units
        chargingManager.printDeviceConsumptions();

        chargingManager.changeDeviceConsumption("Device A", 20);
        chargingManager.printDeviceConsumptions();

        chargingManager.disconnectDevice("Device B");
        chargingManager.printDeviceConsumptions();
   	 } 
   	}


The implementation includes a Device Class that stores device IDs, connection times, and current consumption, allowing for tracking individual device details. The ChargingManager Class manages device connections, disconnections, and consumption changes, using a Queue for FIFO order and a HashMap for quick device lookup. The core methods include connectDevice(), disconnectDevice(), changeDeviceConsumption(), and rebalanceConsumption(), which handle device management and ensure the total consumption does not exceed the system capacity. The consumption management prioritizes devices based on connection time, reducing consumption from the earliest connected devices first, while maintaining a maximum system capacity of 92 units and limiting individual device consumption to 40 units. The demonstration scenario simulates device connections, consumption changes, and disconnections, printing the device consumptions at each step. The implementation offers advantages such as FIFO-based resource allocation, dynamic power management, and the ability to handle multiple edge cases, with potential improvements including more robust error handling, logging, and more sophisticated rebalancing strategies.
