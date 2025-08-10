# LiveKit Swift Voice Agent - Development Setup

## 🎯 iPhone Development Workflow

### Prerequisites
- ✅ Xcode installed
- ✅ LiveKit Cloud account
- ✅ iPhone for testing

### Setup Steps

1. **Configure Xcode Developer Tools**
   ```bash
   sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
   ```

2. **Get LiveKit Sandbox Credentials**
   - Go to [LiveKit Cloud Sandbox](https://cloud.livekit.io/projects/p_/sandbox)
   - Create a Token Server
   - Copy your Sandbox ID

3. **Configure Environment**
   ```bash
   cp MataTalk/.env.example.xcconfig MataTalk/.env.xcconfig
   # Edit .env.xcconfig with your LIVEKIT_SANDBOX_ID
   ```

4. **Open in Xcode**
   ```bash
   open MataTalk.xcodeproj
   ```

### Development Workflow

#### Option A: VS Code + Xcode (Recommended)
- **VS Code**: Code editing, Git, file management
- **Xcode**: Building, debugging, iPhone deployment

#### Option B: Pure Xcode
- All development in Xcode IDE
- Better integrated debugging
- Full iOS Simulator access

### iPhone Testing
1. Connect iPhone via USB
2. Enable Developer Mode on iPhone
3. Trust your Mac in iPhone settings
4. Select your iPhone as target in Xcode
5. Build and run (⌘+R)

### Useful Commands

```bash
# Check project info
xcodebuild -list -project MataTalk.xcodeproj

# Build for simulator
xcodebuild -project MataTalk.xcodeproj -scheme MataTalk -destination 'platform=iOS Simulator,name=iPhone 15 Pro'

# Build for device (requires code signing)
xcodebuild -project MataTalk.xcodeproj -scheme MataTalk -destination 'platform=iOS,id=<device_id>'
```

### Troubleshooting
- Ensure Apple Developer account for device testing
- Configure code signing in Xcode project settings
- Check LiveKit credentials are correctly set
