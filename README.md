# rdp-connect

A simple bash wrapper for macOS that creates RDP connections using Windows App (formerly Microsoft Remote Desktop) with built-in SOCKS5 proxy support.

## Features

- **SSH-style syntax**: Connect using `user@host` format
- **Integrated SSH tunneling**: Automatically creates SSH SOCKS5 tunnels with `--via` option
- **SOCKS5 proxy support**: Automatically tunnels RDP through SOCKS5 proxy (enabled by default)
- **Dynamic port allocation**: Automatically finds available local ports for tunneling
- **Simple .rdp generation**: Creates minimal .rdp files on-the-fly
- **Automatic cleanup**: Removes temporary files and kills tunnels on exit

## Requirements

- macOS
- [Windows App](https://apps.apple.com/us/app/windows-app/id1295203466) (formerly Microsoft Remote Desktop)
- `socat` (for SOCKS5 proxy mode)

### Installing socat

```bash
brew install socat
```

## Installation

1. Download the script:
```bash
curl -O https://raw.githubusercontent.com/Wnt/rdp-connect/main/rdp-connect
chmod +x rdp-connect
```

2. Optionally, move it to your PATH:
```bash
sudo mv rdp-connect /usr/local/bin/
```

## Usage

### Basic Examples

Connect via SOCKS5 proxy (default):
```bash
./rdp-connect jeff@192.168.176.74
```

Connect via SSH tunnel to jumphost:
```bash
./rdp-connect jeff@internal-server -v user@jumphost.com
# or
./rdp-connect jeff@internal-server --via user@jumphost.com
```

Connect in fullscreen mode:
```bash
./rdp-connect jeff@192.168.176.74 -f
```

Connect directly without proxy:
```bash
./rdp-connect jeff@192.168.176.74 --direct
```

Custom SOCKS5 proxy port:
```bash
./rdp-connect jeff@192.168.176.74 --socks-port 1080
```

Traditional syntax (username and host separately):
```bash
./rdp-connect -u admin -h 10.0.0.5 -p 3390
```

## Command-Line Options

### Positional Argument
```
[user@]host              SSH-style connection string (e.g., jeff@192.168.176.74)
```

### Connection Options
```
-h, --host HOSTNAME      Hostname or IP address (default: localhost)
-p, --port PORT          RDP port (default: 3389)
-u, --username USERNAME  Username for RDP connection
-f, --fullscreen         Use fullscreen mode (default: windowed)
```

### SSH Tunnel Options
```
-v, --via [user@]jumphost
                         Create SSH SOCKS5 tunnel via jumphost
```

### SOCKS5 Proxy Options
```
--socks-host HOST        SOCKS5 proxy host (default: 127.0.0.1)
--socks-port PORT        SOCKS5 proxy port (default: 5000)
--direct                 Connect directly without SOCKS5 proxy
```

### Other
```
--help                   Show help message
```

## How It Works

### SOCKS5 Proxy Mode (Default)

1. Finds a random available local port (e.g., 29753)
2. Creates a socat tunnel:
   ```
   socat TCP-LISTEN:29753,reuseaddr,fork SOCKS5:127.0.0.1:192.168.176.74:3389,socksport=5000
   ```
3. Generates a temporary .rdp file pointing to `localhost:29753`
4. Launches Windows App with the .rdp file
5. Keeps the tunnel running until you press Ctrl+C
6. Cleans up the tunnel and temporary files on exit

### Direct Mode

When using `--direct`, the script skips the SOCKS5 tunnel and creates an .rdp file directly to the target host.

## Use Cases

### Connecting Through SSH Tunnel (Integrated)

Use `--via` to automatically create an SSH tunnel and connect:
```bash
./rdp-connect jeff@internal-windows-server.local --via user@jumphost.example.com
```

The script will:
1. Create an SSH SOCKS5 tunnel to the jumphost
2. Set up a socat tunnel through the SSH proxy
3. Connect to the internal Windows server
4. Clean up both tunnels when you're done

### Connecting Through SSH SOCKS5 Tunnel (Manual)

Alternatively, create your own SSH SOCKS5 tunnel first:
```bash
ssh -D 5000 -N user@jumphost.example.com
```

Then connect to a Windows host accessible through the jumphost:
```bash
./rdp-connect jeff@internal-windows-server.local
```

### Using with Custom SOCKS5 Proxy

If you have a SOCKS5 proxy running on a different port:
```bash
./rdp-connect jeff@192.168.1.100 --socks-port 1080
```

### Direct Connections

For direct connections without a proxy:
```bash
./rdp-connect jeff@public-rdp-server.example.com --direct
```

## RDP File Format

The script generates minimal .rdp files with only essential settings:
```
screen mode id:i:1          # 1=windowed, 2=fullscreen
full address:s:localhost:29753
username:s:jeff
```

## Troubleshooting

### "socat is required for SOCKS5 proxy mode"

Install socat:
```bash
brew install socat
```

### "Failed to start SOCKS5 tunnel"

Check that your SOCKS5 proxy is running and accessible:
```bash
# Test with curl
curl --socks5 127.0.0.1:5000 http://example.com
```

### Connection prompts for password every time

This is expected behavior. The script does not store passwords. Windows App will prompt you to enter your password when connecting.

### Windows App not found

Make sure Windows App is installed from the Mac App Store:
https://apps.apple.com/us/app/windows-app/id1295203466

## Tips

- Keep the terminal window open while using the RDP connection in SOCKS5 mode
- Press Ctrl+C in the terminal when you're done to close the tunnel
- The script automatically cleans up temporary files even if interrupted

## License

MIT License - feel free to use and modify as needed.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
