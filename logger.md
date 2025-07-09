# Logger Package

## Overview

The logger package provides a custom `slog` handler with colored console output, JSON attribute handling, and comprehensive OpenTelemetry integration. It wraps Go's structured logging (`slog`) with enhanced formatting and observability features.

## Key Features

- **Custom slog Handler**: Wraps `slog.JSONHandler` with pretty formatting
- **Color-coded Output**: Process and area-specific color coding
- **OpenTelemetry Integration**: Full OTEL standards compliance
- **Thread-safe Operations**: Atomic operations and mutex protection
- **Extensible Configuration**: Option-based configuration pattern

## Code Quality Improvements Applied

### Thread Safety
- **Fixed**: `processColour` global variable now uses atomic operations
- **Added**: Proper mutex protection for shared state

### Type Safety  
- **Fixed**: Replaced panic-prone type assertions with safe checks
- **Added**: Comprehensive error handling with descriptive messages

### Constants
- **Added**: `MaxInlineAttrsLength = 42` for formatting threshold
- **Added**: `JSONIndentSpaces` for consistent indentation
- **Extracted**: All magic numbers into named constants

## OpenTelemetry Integration

### Standards Implemented

#### 1. Trace Context Propagation
- Extracts trace ID, span ID from context
- Includes trace correlation in log records  
- Supports W3C Trace Context headers
- Displays abbreviated trace ID in output

#### 2. Structured Attributes
- Uses semantic conventions for common attributes
- Supports resource attributes (service.name, service.version)
- Adds instrumentation scope information
- Environment-based configuration

#### 3. Log Record Fields
- **Timestamp**: RFC3339 format (`timestamp`)
- **Severity Mapping**: OTEL severity numbers
  - TRACE = 1, DEBUG = 5, INFO = 9, WARN = 13, ERROR = 17, FATAL = 21
- **Body Field**: Log message content
- **Attributes Map**: Structured data

#### 4. Integration Points
- Bridge with OpenTelemetry Logger Provider
- Support for OTLP export format
- Correlation with traces and metrics

### OTEL Field Names

```go
const (
    OTELTraceIDKey      = "trace_id"
    OTELSpanIDKey       = "span_id" 
    OTELTraceFlagsKey   = "trace_flags"
    OTELServiceNameKey  = "service.name"
    OTELServiceVersionKey = "service.version"
    OTELResourceKey     = "resource"
    OTELScopeNameKey    = "scope.name"
    OTELScopeVersionKey = "scope.version"
    OTELTimestampKey    = "timestamp"
    OTELSeverityTextKey = "severity_text"
    OTELSeverityNumberKey = "severity_number"
    OTELBodyKey         = "body"
    OTELAttributesKey   = "attributes"
)
```

## API Reference

### Core Functions

#### Handler Creation
```go
// Standard handler
NewHandler(opts *slog.HandlerOptions, params PrettyLogParams, options ...Option) *Handler

// OpenTelemetry-enabled handler  
NewOTELHandler(opts *slog.HandlerOptions, params PrettyLogParams, options ...Option) *Handler
```

#### Environment-based Loggers
```go
// Standard logger from environment
CreateLoggerFromEnv(out *os.File, colour string) *slog.Logger

// OTEL logger from environment
CreateOTELLoggerFromEnv(out *os.File, colour string) *slog.Logger
```

### Configuration Options

```go
// Standard options
WithTimestamp()                    // Add timestamps
WithDestinationWriter(io.Writer)   // Set output destination
WithColour()                      // Enable color output
WithOutputEmptyAttrs()            // Show empty attributes

// OpenTelemetry options
WithOpenTelemetry()               // Enable OTEL integration
WithResource(*resource.Resource)   // Set OTEL resource
WithInstrumentationScope(name, version string) // Set scope info
```

### Utility Functions

```go
// Resource creation
NewDefaultResource() *resource.Resource

// Trace context extraction
extractTraceContext(ctx context.Context) (traceID, spanID, traceFlags string)

// Level mapping
mapSlogLevelToOTELSeverity(level slog.Level) int
```

## Usage Examples

### Basic Usage
```go
// Create basic logger
logger := CreateLoggerFromEnv(nil, "blue")
logger.Info("Hello World", "key", "value")
```

### OpenTelemetry Usage
```go
// Create OTEL logger
logger := CreateOTELLoggerFromEnv(nil, "blue")

// With trace context
ctx := context.WithValue(context.Background(), "trace", traceInfo)
logger.InfoContext(ctx, "Request processed", "user_id", 123)
```

### Custom Handler
```go
handler := NewOTELHandler(
    &slog.HandlerOptions{Level: slog.LevelInfo},
    NewParams(os.Stdout),
    WithInstrumentationScope("my-service", "2.0.0"),
    WithResource(customResource),
)
logger := slog.New(handler)
```

### Programmatic Logger Setup
```go
prettyHandler := NewHandler(&slog.HandlerOptions{
    Level:       LevelTrace,
    AddSource:   false,
    ReplaceAttr: nil,
}, NewParams(os.Stdout))

logger := slog.New(prettyHandler)
slog.SetDefault(logger)
```

## Environment Variables

### OpenTelemetry Configuration
- `OTEL_SERVICE_NAME` - Service name (default: "easy-test")
- `OTEL_SERVICE_VERSION` - Service version (default: "1.0.0") 
- `OTEL_LOGS_ENABLED=true` - Enable OTEL features
- `DEPLOYMENT_ENVIRONMENT` - Environment (dev/staging/prod)

### Logger Configuration  
- `DEBUG_LOG` - Log file path (default: stderr)
- `DEBUG_TYPE=pretty` - Enable pretty formatting
- `NO_PRETTY_LOGGER` - Disable pretty formatting

## Log Levels

### Standard Levels
- `LevelTrace` - Detailed execution information
- `slog.LevelDebug` - Debug information
- `slog.LevelInfo` - General information
- `slog.LevelWarn` - Warning conditions
- `slog.LevelError` - Error conditions
- `LevelFatal` - Fatal error conditions

### Color Mapping
- **TRACE/DEBUG**: Light gray
- **INFO**: Cyan  
- **WARN**: Light yellow
- **ERROR**: Red
- **FATAL**: Magenta

## Output Format

### Standard Format
```
process:area [trace_id] LEVEL message key=value key2=value2
```

### OpenTelemetry Enhanced Format
```
process:area [12345678] INFO message service=my-service key=value
```

### JSON Attributes (when exceeding inline threshold)
```
process:area INFO message {
  "key": "value",
  "trace_id": "1234567890abcdef",
  "service.name": "my-service"
}
```

## Dependencies

### Required Packages
```go
import (
    "go.opentelemetry.io/otel/trace"
    "go.opentelemetry.io/otel/attribute" 
    "go.opentelemetry.io/otel/sdk/resource"
    "go.opentelemetry.io/otel/semconv/v1.21.0"
)
```

## Migration Guide

### From Original to Fixed Version
1. **Thread Safety**: No code changes needed - automatic
2. **Type Safety**: Error handling improved - check return values
3. **Constants**: Use named constants instead of magic numbers

### Enabling OpenTelemetry
1. **Environment**: Set `OTEL_LOGS_ENABLED=true`
2. **Code**: Use `CreateOTELLoggerFromEnv()` instead of `CreateLoggerFromEnv()`
3. **Custom**: Use `NewOTELHandler()` for advanced configuration

### Backward Compatibility
- All existing APIs remain functional
- OpenTelemetry features are opt-in
- Default behavior unchanged when OTEL disabled

## Best Practices

### Performance
- Use appropriate log levels to avoid overhead
- Leverage structured logging with key-value pairs
- Consider async logging for high-throughput scenarios

### Observability
- Always include trace context in request handlers
- Use consistent service names across components
- Include relevant business context in log attributes

### Security
- Never log sensitive information (passwords, tokens)
- Use structured fields to avoid log injection
- Sanitize user input before logging

## Troubleshooting

### Common Issues
1. **Missing Trace Context**: Ensure context propagation in handlers
2. **Resource Creation Fails**: Check environment variables and fallback
3. **Color Output Issues**: Verify terminal color support
4. **Performance Impact**: Consider log level filtering

### Debug Environment Variables
```bash
export DEBUG_TYPE=pretty
export OTEL_LOGS_ENABLED=true
export OTEL_SERVICE_NAME=my-service
export DEBUG_LOG=/tmp/debug.log
```