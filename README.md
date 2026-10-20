# Instrumentasi_Medis
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ESP32 Bluetooth Dashboard</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; background-color: #f4f4f9; }
        .card { background: white; padding: 20px; border-radius: 10px; display: inline-block; box-shadow: 0px 4px 10px rgba(0,0,0,0.1); }
        button { padding: 12px 24px; font-size: 16px; cursor: pointer; background-color: #007BFF; color: white; border: none; border-radius: 5px; }
        button:hover { background-color: #0056b3; }
        .data-display { margin-top: 20px; font-size: 22px; }
    </style>
</head>
<body>

    <div class="card">
        <h1>ESP32 Live Weather</h1>
        <button id="connectBtn">Connect to ESP32</button>
        
        <div class="data-display">
            <p>Temperature: <span id="temp">--</span> &deg;C</p>
            <p>Humidity: <span id="hum">--</span> %</p>
        </div>
    </div>

    <script>
        const serviceUUID = "4fafc201-1fb5-459e-8fcc-c5c9c331914b";
        const characteristicUUID = "beb5483e-36e1-4688-b7f5-ea07361b26a1";

        const connectBtn = document.getElementById('connectBtn');
        const tempSpan = document.getElementById('temp');
        const humSpan = document.getElementById('hum');

        connectBtn.addEventListener('click', async () => {
            try {
                // 1. Request Bluetooth device filtered by name
                console.log('Requesting Bluetooth Device...');
                const device = await navigator.bluetooth.requestDevice({
                    filters: [{ name: 'ESP32_Sensor' }],
                    optionalServices: [serviceUUID]
                });

                // 2. Connect to the device's GATT server
                console.log('Connecting to GATT Server...');
                const server = await device.gatt.connect();

                // 3. Get the custom service
                console.log('Getting Service...');
                const service = await server.getPrimaryService(serviceUUID);

                // 4. Get the characteristic
                console.log('Getting Characteristic...');
                const characteristic = await service.getCharacteristic(characteristicUUID);

                // 5. Start listening for live notifications
                console.log('Starting Notifications...');
                await characteristic.startNotifications();
                
                characteristic.addEventListener('characteristicvaluechanged', (event) => {
                    const value = new TextDecoder().decode(event.target.value);
                    
                    // Parse the "temp,humidity" string payload
                    const parts = value.split(',');
                    if (parts.length === 2) {
                        tempSpan.textContent = parts[0];
                        humSpan.textContent = parts[1];
                    }
                });

                // Update UI upon successful connection
                connectBtn.textContent = 'Connected';
                connectBtn.style.backgroundColor = '#28a745';
                connectBtn.disabled = true;

            } catch (error) {
                console.error('Connection failed: ', error);
                alert('Connection failed or canceled: ' + error);
            }
        });
    </script>
</body>
</html>
