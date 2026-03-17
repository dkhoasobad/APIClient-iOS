# APIClient-iOS

> Static library for integrating [ServerKey](https://vuxuancuong.dev/) license validation into jailbroken iOS tweaks.

<p align="center">
  <img src="https://github.com/dkhoasobad/APIClient-iOS/blob/main/Image/Demo.jpg" width="300" alt="License Gate UI Example"/>
</p>

## Features

- One-line license gate integration
- Automatic license UI presentation
- Periodic license re-verification
- Secure keychain storage
- SSL-pinned API communication
- Fully obfuscated — no readable strings in binary (IDA/Hopper safe)

## Requirements

| Requirement | Version |
|---|---|
| iOS Deployment Target | 14.0+ |
| Architecture | arm64 |
| Build System | [Theos](https://theos.dev/) |

## Installation

1. Download the latest release:
   - `libAPIClient.a`
   - `APIClient.h`

2. Copy both files into your Theos project:
```
YourTweak/
├── Tweak.xm
├── Makefile
├── APIClient.h       ← header
└── lib/
    └── libAPIClient.a  ← static library
```

3. Update your `Makefile`:
```makefile
# Link the static library
YourTweak_LDFLAGS += -L$(THEOS_PROJECT_DIR)/lib -lAPIClient

# Link required frameworks
YourTweak_FRAMEWORKS = UIKit Foundation Security

# Link OpenSSL (required — get from your provider)
YourTweak_LDFLAGS += -L$(THEOS_PROJECT_DIR)/lib -lssl -lcrypto
```

> **Note:** You must provide your own `libssl.a` and `libcrypto.a` built for arm64 iOS.

## Usage

### Basic Integration

```objc
#import "APIClient.h"

%ctor {
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC),
        dispatch_get_main_queue(), ^{

        APIClient *API = [[APIClient alloc] init];
        [API setPackageHash:@"YOUR_PACKAGE_HASH"];
        [API paid:^{
            // This code ONLY runs when the license is valid
            NSLog(@"License verified! Paid features unlocked.");
        }];

    });
}
```

### How It Works

1. **`setPackageHash:`** — Set your package token from the [ServerKey Dashboard](https://vuxuancuong.dev/). Must be called before `paid:`.

2. **`paid:`** — Gates your code behind license validation:
   - **Valid license on device** → block runs immediately
   - **No license** → license UI is presented, block runs after successful activation
   - **Validation fails** → block never runs

### API Reference

```objc
@interface APIClient : NSObject

/// Set the package hash token. MUST be called before paid:.
- (void)setPackageHash:(NSString *)hash;

/// Gate a block behind license validation.
- (void)paid:(void (^)(void))paidBlock;

@end
```

## Screenshots

| License Gate | Activation Success |
|:---:|:---:|
| <img src="https://github.com/dkhoasobad/APIClient-iOS/blob/main/Image/Demo.jpg" width="250"/> | <img src="https://github.com/dkhoasobad/APIClient-iOS/blob/main/Image/Demo1.jpg" width="250"/> |

> Add your screenshots to a `screenshots/` folder in the repo.

## Building from Source

```bash
cd ios-license-client
make clean && make
```

Output will be in `output/`:
```
output/
├── libAPIClient.a    # Static library (arm64)
└── APIClient.h       # Public header
```

## Troubleshooting

### `ld: archive has no table of contents`
Your toolchain's `ar`/`ranlib` may not support arm64. Rebuild with Theos on macOS or ensure your cross-compile toolchain includes `arm-apple-darwin` binutils.

### `Undefined symbols: _OBJC_CLASS_$_FCUUID`
FCUUID is bundled inside `libAPIClient.a`. Run `make clean && make` to ensure all object files are included.

### `ld: warning: built for newer iOS version`
Safe to ignore. OpenSSL was compiled with a newer SDK but is backwards compatible.

## License

Private — All rights reserved.
Distributed exclusively through [ServerKey](https://vuxuancuong.dev/).

---

<p align="center">
  <b>ServerKey</b> — License management for iOS tweaks<br/>
  <a href="https://vuxuancuong.dev/">https://vuxuancuong.dev/</a>
</p>
