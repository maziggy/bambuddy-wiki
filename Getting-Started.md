# Getting Started

This guide will help you set up your first printer and start using Bambuddy.

## Prerequisites

Before adding a printer, ensure:
- Bambuddy is running (see [[Installation]])
- Your printer is powered on and connected to your network
- You have access to your printer's settings

---

## Enabling LAN Mode on Your Printer

Bambuddy connects to printers via LAN Mode. Here's how to enable it:

### Step 1: Enable Developer Mode (if required)
Some printer models require Developer Mode to be enabled first:
1. On your printer's touchscreen, go to **Settings**
2. Navigate to **General** or **About**
3. Look for **Developer Mode** and enable it

### Step 2: Enable LAN Mode
1. Go to **Settings** > **Network** > **LAN Mode**
2. Toggle **LAN Mode** to **ON**
3. Note down the **Access Code** displayed (8 characters)

### Step 3: Find Your Printer's Information
You'll need these details:
- **IP Address**: Found in **Settings** > **Network**
- **Serial Number**: Found in **Settings** > **Device Info**
- **Access Code**: Shown when LAN Mode is enabled

> **Tip**: The access code changes every time you toggle LAN Mode off and on.

---

## Adding Your First Printer

1. Open Bambuddy in your browser (default: http://localhost:8000)
2. Go to the **Printers** page
3. Click **Add Printer**
4. Enter the following information:
   - **Name**: A friendly name (e.g., "Workshop X1C")
   - **IP Address**: Your printer's local IP
   - **Access Code**: The 8-character code from LAN Mode
   - **Serial Number**: Your printer's serial number
5. Click **Save**

The printer should connect automatically. You'll see:
- A **green indicator** when connected
- Real-time status updates (temperatures, print progress, etc.)

---

## Understanding the Interface

### Printers Page
- **Printer Cards**: Show real-time status for each printer
- **HMS Status**: Health Management System indicator (green = OK)
- **Temperature Readouts**: Nozzle, bed, and chamber temperatures
- **Print Progress**: Current layer, time remaining, filament usage

### Archives Page
- **Archive Cards**: All completed prints with thumbnails
- **Filters**: Sort by printer, date, status, collections
- **Search**: Find prints by name
- **Context Menu**: Right-click for actions (re-print, delete, etc.)

### Statistics Page
- **Drag-and-drop Widgets**: Customize your dashboard
- **Print Success Rate**: Track reliability
- **Filament Usage**: Monitor material consumption
- **Cost Tracking**: Calculate printing costs

---

## Your First Archived Print

Once your printer is connected, Bambuddy automatically archives completed prints:

1. Start a print on your Bambu Lab printer (via Bambu Studio, Handy, or the printer itself)
2. Bambuddy detects the print and monitors progress
3. When the print completes:
   - The 3MF file is downloaded via FTP
   - Metadata is extracted (layers, filament, temperatures, etc.)
   - A thumbnail is generated
   - The print appears in your Archives

### What Gets Archived
- **3MF File**: The complete print file
- **Thumbnail**: Preview image from the slicer
- **Metadata**: Print time, filament usage, layer count, temperatures
- **Finish Photo**: Automatic camera capture (if enabled)
- **Print Result**: Success, failed, or stopped status

---

## Keyboard Shortcuts

Navigate quickly with keyboard shortcuts:

| Key | Action |
|-----|--------|
| `1` | Go to Printers |
| `2` | Go to Archives |
| `3` | Go to Statistics |
| `4` | Go to Cloud Profiles |
| `5` | Go to Settings |
| `?` | Show keyboard shortcuts |

---

## Mobile Access

Bambuddy works on phones and tablets:

- **Hamburger Menu**: Tap the menu icon to open navigation
- **Touch-Friendly**: All controls are sized for easy tapping
- **Responsive Layout**: Pages adapt to smaller screens
- **Context Menus**: Tap the three-dot icon on cards for actions

---

## Next Steps

Now that you're set up, explore these features:

- **[[Features Print Queue]]** - Schedule prints for later
- **[[Features Smart Plugs]]** - Automate power control
- **[[Features Notifications]]** - Get alerts on your phone
- **[[Features Spoolman]]** - Track filament inventory

---

## Having Issues?

- **Printer won't connect?** See [[Troubleshooting#printer-connection-issues]]
- **Prints not archiving?** See [[Troubleshooting#archiving-issues]]
- **Need help?** Open an [issue on GitHub](https://github.com/maziggy/bambuddy/issues)
