# Documentation

## Adding a marker from Unity

Example of a **SendMarker** function to send a marker in the session csv file (Marker column)

**Port :** by default, the port is set to 1099 (vous pouvew changer le port dans le fichier C:/Kaptics/udpport.txt)

**IpAddress :** if your Unity project and Kaptics are running on the same machine, leave 127.0.0.1 otherwise enter the IP address of the PC running Kaptics
```
private void SendMarker(int Port = 1099, string IpAddress = "127.0.0.1")
{            
    UdpClient udp = new UdpClient();
    IPEndPoint ep = new IPEndPoint(IPAddress.Parse(IpAddress), Port);

    udp.Connect(ep);

    byte[] emptyMessage = new byte[0];
    udp.Send(emptyMessage, emptyMessage.Length);

    udp.Close();
}
```
