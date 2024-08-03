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
