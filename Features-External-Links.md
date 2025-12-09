# External Sidebar Links

Add custom links to external tools directly in the sidebar navigation.

## Overview

External links let you:
- Add quick access to other tools
- Customize the sidebar with your frequently used services
- Embed external pages within Bambuddy

---

## Use Cases

Quick access to:
- **OctoPrint** or **Mainsail** (other print managers)
- **Spoolman** (filament inventory)
- **Obico** (AI print monitoring)
- **Home Assistant** (home automation)
- Documentation or reference sites
- Internal dashboards

---

## Adding External Links

1. Go to **Settings**
2. Scroll to **Sidebar Links**
3. Click **Add Link**
4. Configure:
   - **Name**: Display name for the link
   - **URL**: Full URL to the external site
   - **Icon**: Choose icon type
5. Click **Save**

---

## Link Configuration

### Name
- Display text shown in sidebar
- Keep it short (1-2 words)
- Examples: "Spoolman", "OctoPrint", "Docs"

### URL
- Full URL including protocol
- Examples:
  - `http://192.168.1.100:7912` (Spoolman)
  - `https://docs.example.com`
  - `http://octopi.local`

### Icon Options

**Built-in Icons**
Choose from common icons:
- Link icon
- External link icon
- Home icon
- Settings icon
- And more...

**Custom Icons**
Upload your own:
1. Select "Custom Icon"
2. Upload an SVG file
3. Icon is stored and displayed

---

## Managing Links

### Reordering
Drag links to change their position:
1. Hover over the link
2. Click and hold the grip handle
3. Drag to new position
4. Release to drop

Links can be mixed with internal navigation items.

### Editing
1. Click the pencil icon on a link
2. Modify settings
3. Click **Save**

### Deleting
1. Click the trash icon on a link
2. Confirm deletion

---

## How Links Open

### Embedded Mode (Default)
External pages open in an iframe within Bambuddy:
- Stays within Bambuddy interface
- Quick switching between tools
- Sidebar remains accessible

### New Tab
Some sites require opening in a new tab:
- Sites that block iframe embedding
- Complex applications
- When you need full browser features

---

## Iframe Limitations

Some websites block iframe embedding for security:

### Common Restrictions
- `X-Frame-Options` header set to DENY
- Content Security Policy restrictions
- Same-origin policy violations

### Affected Sites
Sites that typically don't work in iframes:
- Google services
- Social media sites
- Banking/financial sites
- Some cloud dashboards

### Workaround
If a site doesn't load in the iframe:
1. Right-click the link
2. Select "Open in New Tab"
3. Or configure the link to always open externally

---

## Icon Upload

### Supported Formats
- SVG (recommended)
- PNG
- JPG

### Icon Guidelines
- Square aspect ratio works best
- Keep file size small (<50KB)
- Simple designs show better at sidebar size
- SVG scales best at any size

### Finding Icons
Sources for icons:
- [Lucide Icons](https://lucide.dev/)
- [Simple Icons](https://simpleicons.org/)
- [Heroicons](https://heroicons.com/)
- Product brand guidelines (for official logos)

---

## Examples

### Spoolman
```
Name: Spoolman
URL: http://192.168.1.100:7912
Icon: Built-in database icon
```

### Home Assistant
```
Name: Home
URL: http://homeassistant.local:8123
Icon: Custom (Home Assistant logo SVG)
```

### Documentation
```
Name: Docs
URL: https://wiki.example.com
Icon: Built-in book icon
```

---

## Tips

- Keep link names short for cleaner sidebar
- Use custom icons for brand recognition
- Test iframe embedding before relying on it
- Group related links by ordering them together
- Remove unused links to keep sidebar clean
