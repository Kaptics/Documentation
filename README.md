# Documentation

## Adding a marker from Unity

Example of a **SendMarker** function to send a marker in the session csv file (Marker column)

**Port :** by default, the port is set to 1099 (You can change the port in the file C:/Kaptics/udpport.txt)

**IpAddress :** if your Unity project and Kaptics are running on the same machine, leave 127.0.0.1 otherwise enter the IP address of the PC running Kaptics

**MarkerValue :** if you want to force a value for the marker, the value will increment automatically if not specified.
```
using System.Net;
using System.Net.Sockets;
using System.Text;

private void SendMarker(int Port = 1099, string IpAddress = "127.0.0.1", int MarkerValue = -1)
{
    UdpClient udp = new UdpClient();
    IPEndPoint ep = new IPEndPoint(IPAddress.Parse(IpAddress), Port);

    string request = "PostMarker";
    if (MarkerValue != -1)
    {
        request = $"{request}:{MarkerValue}";
    }

    udp.Connect(ep);

    byte[] message = Encoding.Unicode.GetBytes(request);
    udp.Send(message, message.Length);

    udp.Close();
}
```

## Read data in your application

You can use the brainflow library to read headset data into your own application.

Example of reading data in C#:
```
using System;
using brainflow;
using brainflow.math;

namespace Kaptics
{
    class GetBoardData
    {
        static void Main (string[] args)
        {
            BoardShim.enable_dev_board_logger();
            BrainFlowInputParams input_params = new BrainFlowInputParams();

            int board_id = parse_args(args, input_params);

            BoardShim board_shim = new BoardShim(board_id, input_params);
            board_shim.prepare_session();
            board_shim.start_stream();
            System.Threading.Thread.Sleep(5000);
            board_shim.stop_stream();
            double[,] unprocessed_data = board_shim.get_current_board_data(20);
            int[] eeg_channels = BoardShim.get_eeg_channels(board_id);
            foreach (var index in eeg_channels)
                Console.WriteLine ("[{0}]", string.Join (", ", unprocessed_data.GetRow (index)));
            board_shim.release_session ();
        }

        static int parse_args (string[] args, BrainFlowInputParams input_params)
        {
            int board_id = (int)BoardIds.CYTON_DAISY_WIFI_BOARD;

            for (int i = 0; i < args.Length; i++)
            {
                if (args[i].Equals ("--ip-address"))
                {
                    input_params.ip_address = args[i + 1];
                }
                if (args[i].Equals ("--mac-address"))
                {
                    input_params.mac_address = args[i + 1];
                }
                if (args[i].Equals ("--serial-port"))
                {
                    input_params.serial_port = args[i + 1];
                }
                if (args[i].Equals ("--other-info"))
                {
                    input_params.other_info = args[i + 1];
                }
                if (args[i].Equals ("--ip-port"))
                {
                    input_params.ip_port = Convert.ToInt32 (args[i + 1]);
                }
                if (args[i].Equals ("--ip-protocol"))
                {
                    input_params.ip_protocol = Convert.ToInt32 (args[i + 1]);
                }
                if (args[i].Equals ("--board-id"))
                {
                    board_id = Convert.ToInt32 (args[i + 1]);
                }
                if (args[i].Equals ("--timeout"))
                {
                    input_params.timeout = Convert.ToInt32 (args[i + 1]);
                }
                if (args[i].Equals ("--serial-number"))
                {
                    input_params.serial_number = args[i + 1];
                }
                if (args[i].Equals ("--file"))
                {
                    input_params.file = args[i + 1];
                }
            }
            return board_id;
        }
    }
}
```

## Lab Streaming Layer

Example of a python script to read headset data in LSL:
```
import socket
from pylsl import StreamInfo, StreamOutlet
import struct

# WiFi board connection settings
ip = "192.168.4.1" 
port = 12345
buffer_size = 512

# LSL stream configuration
n_channels = 16
sampling_rate = 250

info = StreamInfo('Kaptics', 'EEG', n_channels, sampling_rate, 'float32', 'Kaptics')
outlet = StreamOutlet(info)

# Connecting to the WiFi board
print(f"Connecting to {ip}:{port}...")
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((ip, port))
print("Connected to the WiFi board.")

try:
    while True:
        # Read raw data from the board
        data = sock.recv(buffer_size)
        
        # Process data (convert to EEG values)
        # EEG data is typically sent in packets of 66 bytes for 16 channels.
        
        if len(data) >= 66: 
            # Decode the data to obtain EEG values
            eeg_data = struct.unpack('<16f', data[:66])
            
            # Send data into LSL stream
            outlet.push_sample(eeg_data)
            print("Data sent to LSL:", eeg_data)
            
except KeyboardInterrupt:
    print("Stopped data reading.")
finally:
    sock.close()
    print("TCP connection closed.")
```

**Code Explanation**

**TCP Connection:** The script establishes a connection to the WiFi board using the IP address and port.

**Data Reception:** sock.recv(buffer_size) receives raw EEG data from the board.

**Data Decoding:** The data is converted to EEG values (in float format) using the struct board to interpret it properly.

**Data Streaming via LSL:** EEG data is pushed into an LSL stream, allowing real-time access from any compatible LSL program or framework.
