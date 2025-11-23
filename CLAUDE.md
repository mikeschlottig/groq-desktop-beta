# CLAUDE.md - AI Assistant Guide for Groq Desktop

## Project Overview

Groq Desktop is an Electron-based desktop application that provides a chat interface for Groq's LLM API with support for Model Context Protocol (MCP) servers. It enables users to interact with Groq's fast inference models while leveraging external tools through MCP.

**Key Features**:
- Chat interface with streaming responses
- Image/vision support for compatible models
- MCP server integration for tool use
- Multi-platform support (macOS, Windows, Linux)
- Global hotkey for context capture

## Tech Stack

- **Framework**: Electron 37.0.0
- **Frontend**: React 19.0.0, Vite 6.2.6
- **Styling**: TailwindCSS 3.3.3, Radix UI components
- **API Client**: groq-sdk 0.16.0
- **MCP Integration**: @modelcontextprotocol/sdk 1.7.0
- **Package Manager**: pnpm 10.9.0
- **Build Tool**: electron-builder 24.13.3
- **Code Quality**: ESLint 9, Prettier 3.6.0

## Project Structure

```
groq-desktop-beta/
├── electron/                 # Main process code
│   ├── main.js              # App entry point
│   ├── preload.js           # Preload script for IPC
│   ├── chatHandler.js       # Chat streaming logic
│   ├── mcpManager.js        # MCP server connections
│   ├── settingsManager.js   # User settings persistence
│   ├── authManager.js       # OAuth handling for MCP
│   ├── commandResolver.js   # Platform-specific command resolution
│   ├── toolHandler.js       # MCP tool execution
│   ├── windowManager.js     # Window lifecycle
│   ├── popupWindow.js       # Context capture popup
│   ├── contextCapture.js    # Global hotkey capture
│   └── scripts/             # Platform-specific runner scripts
├── src/
│   └── renderer/            # React frontend
│       ├── App.jsx          # Main application component
│       ├── main.jsx         # React entry point
│       ├── index.css        # Global styles
│       ├── components/      # UI components
│       │   ├── ChatInput.jsx
│       │   ├── MessageList.jsx
│       │   ├── ToolsPanel.jsx
│       │   └── ui/          # Reusable UI primitives
│       ├── context/         # React context providers
│       └── lib/             # Utilities
├── shared/                  # Code shared between main/renderer
│   └── models.js            # Model configurations
├── public/                  # Static assets
├── .github/workflows/       # CI/CD pipelines
├── test-*.js               # Test scripts
└── electron-builder.yml    # Build configuration
```

## Key Files

| File | Purpose |
|------|---------|
| `electron/main.js` | Application entry, IPC handlers, lifecycle |
| `electron/chatHandler.js` | Streaming chat with Groq API, retry logic |
| `electron/mcpManager.js` | MCP server connections (stdio, SSE, HTTP) |
| `src/renderer/App.jsx` | Main React component, chat UI logic |
| `shared/models.js` | Model definitions and context sizes |
| `electron/preload.js` | Secure bridge between main/renderer |
| `package.json` | Dependencies and build scripts |
| `electron-builder.yml` | Distribution build configuration |

## Development Setup

### Prerequisites
- Node.js 18+
- pnpm (will be installed via corepack if needed)

### Quick Start

```bash
# Install dependencies
pnpm install

# If electron fails to install, approve build scripts
pnpm approve-builds

# Start development server
pnpm dev
```

### Configuration

1. Launch the app and go to Settings
2. Add your Groq API key (get one at https://console.groq.com/keys)
3. Configure MCP servers if needed

Settings are stored in:
- macOS: `~/Library/Application Support/groq-desktop-app/settings.json`
- Windows: `%APPDATA%/groq-desktop-app/settings.json`
- Linux: `~/.config/groq-desktop-app/settings.json`

## Common Commands

```bash
# Development
pnpm dev                    # Start dev server with hot reload
pnpm dev:vite              # Start only Vite dev server
pnpm dev:electron          # Start only Electron

# Building
pnpm build                 # Build Vite frontend
pnpm dist                  # Build distributable for current platform
pnpm dist:mac              # Build macOS .dmg
pnpm dist:win              # Build Windows installer
pnpm dist:linux            # Build Linux AppImage/deb/rpm

# Testing
pnpm test:platforms        # Run cross-platform tests
pnpm test:paths            # Test path resolution
node test-resolver.js      # Test command resolution

# Code Quality
npx eslint .               # Run ESLint
npx prettier --write .     # Format code
```

## Testing

### Current Test Setup
The project uses ad-hoc Node.js test scripts rather than a formal test framework:

- `test-paths.js` - Tests path handling
- `test-resolver.js` - Tests command resolution
- `test-platform-detection.js` - Tests platform detection
- `test-cross-platform.sh` - Shell script for platform tests
- `test-popup-window.js` - Tests popup window functionality

### Running Tests

```bash
# Run all platform tests
pnpm test:platforms

# Run individual test files
node test-paths.js
node test-resolver.js
```

## Code Conventions

### JavaScript Style
- ESLint with recommended rules
- Prettier for formatting
- CommonJS modules in Electron main process
- ES modules in renderer (JSX)

### Component Pattern
React functional components with hooks:
```javascript
function MyComponent({ prop1, prop2 }) {
  const [state, setState] = useState(initialValue);

  useEffect(() => {
    // Side effects
  }, [dependencies]);

  return (
    <div className="tailwind-classes">
      {/* JSX */}
    </div>
  );
}
```

### IPC Communication Pattern
Main process handlers:
```javascript
ipcMain.handle('channel-name', async (event, arg) => {
  // Handle and return result
  return result;
});
```

Renderer invocation:
```javascript
const result = await window.electron.channelName(arg);
```

### File Naming
- Components: PascalCase (e.g., `ChatInput.jsx`)
- Utilities: camelCase (e.g., `messageUtils.js`)
- Test files: `test-*.js`

## API Structure

### Groq API Integration
- Chat completions with streaming
- Support for tool calls (function calling)
- Vision/image support for compatible models
- Compound model support with reasoning

### MCP Server Support
Three transport types:
- **stdio**: Local process communication
- **sse**: Server-Sent Events
- **streamableHttp**: HTTP streaming

## When Making Changes

1. **Follow existing patterns** - Check similar code in the same directory
2. **Test on multiple platforms** - The app targets Windows, macOS, and Linux
3. **Handle errors gracefully** - Use try/catch and show user-friendly messages
4. **Update IPC handlers** - Changes to main/renderer communication need both sides
5. **Consider MCP compatibility** - Tool changes may affect MCP server integrations
6. **Run linting** - `npx eslint .` before committing

### Adding New Features
1. Add IPC handler in `electron/main.js` or relevant manager
2. Expose through `electron/preload.js`
3. Use in renderer via `window.electron.*`
4. Update settings if needed via `settingsManager.js`

## Issues & Recommendations

### Critical Issues

1. **No Formal Test Framework**
   - Only ad-hoc test scripts exist
   - No unit tests for React components
   - No integration tests
   - **Recommendation**: Add Vitest or Jest with React Testing Library

2. **CI/CD Only Builds macOS**
   - `.github/workflows/build-macos.yml` only builds for macOS
   - Windows and Linux builds are not automated
   - **Recommendation**: Add workflows for Windows and Linux builds

3. **Large Component Files**
   - `App.jsx` is 1300+ lines with mixed concerns
   - Difficult to maintain and test
   - **Recommendation**: Extract chat logic into custom hooks, split into smaller components

### Security Concerns

4. **API Key Storage**
   - API keys stored in plain text in `settings.json`
   - **Recommendation**: Use system keychain (keytar) for sensitive credentials

5. **No Content Security Policy**
   - Preload script doesn't enforce strict CSP
   - **Recommendation**: Add CSP headers to prevent XSS

### Code Quality Issues

6. **No Type Safety**
   - No TypeScript or PropTypes
   - Makes refactoring risky
   - **Recommendation**: Add TypeScript or at minimum PropTypes for components

7. **Missing Error Boundaries**
   - React errors can crash the entire UI
   - **Recommendation**: Add Error Boundaries around major components

8. **Incomplete Package Metadata**
   - `author` field is empty in package.json
   - **Recommendation**: Add author and repository information

### Documentation Gaps

9. **Missing Contributing Guide**
   - No CONTRIBUTING.md
   - No development workflow documentation
   - **Recommendation**: Add contribution guidelines

10. **No Architecture Documentation**
    - Complex IPC communication patterns undocumented
    - MCP integration patterns not explained
    - **Recommendation**: Add architecture diagrams and flow documentation

### Dependency Issues

11. **No Dependency Auditing**
    - No `npm audit` in CI
    - No automated security updates
    - **Recommendation**: Add security scanning workflow

12. **No Lock File Validation**
    - Both `package-lock.json` and `pnpm-lock.yaml` exist
    - Can cause confusion
    - **Recommendation**: Remove `package-lock.json`, use only pnpm

### Performance Considerations

13. **No Code Splitting**
    - Entire app loads at once
    - **Recommendation**: Implement lazy loading for settings, tools panel

14. **Large Bundle Size**
    - KaTeX and syntax highlighting included
    - **Recommendation**: Consider lazy loading these heavy dependencies

## Environment Variables

Key settings (configured via app Settings UI):
- `GROQ_API_KEY` - Required for API access
- Custom model configurations
- MCP server definitions
- Temperature and top_p parameters

## External Resources

- Groq Console: https://console.groq.com
- MCP Protocol: https://modelcontextprotocol.io
- Demo: https://bottlecrm.io/
