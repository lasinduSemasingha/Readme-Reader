# Fair Auctioneers - PWA Installation System Documentation

## 📱 Overview
This document contains the complete implementation of the automatic PWA (Progressive Web App) installation system for Fair Auctioneers. The system provides a beautiful, two-step confirmation process that triggers automatic browser installation with your company logo.

---

## 🎯 Features

### ✅ What This System Does:
1. **Custom Branded Install Modal** - Shows company logo and benefits
2. **Two-Step Confirmation** - Prevents accidental installations
3. **Automatic Browser Trigger** - Automatically opens browser install dialog
4. **Smart Fallbacks** - Handles iOS, Safari, and unsupported browsers
5. **Toast Notifications** - Success/error feedback to users
6. **Company Logo Integration** - Your logo appears on home screen after install
7. **Layered Z-Index System** - Proper modal stacking with enhanced visibility

---

## 📂 File Structure

```
/lib/pwa.ts                          # Core PWA utility functions
/components/InstallPromptModal.tsx   # Custom install modal component
/components/Navbar.tsx               # Installation trigger (navbar button)
/public/manifest.json                # PWA configuration (logo, name, icons)
/public/sw.js                        # Service Worker (optional)
```

---

## 🔧 Complete Code Implementation

### 1. PWA Utility Functions (`/lib/pwa.ts`)

```typescript
// PWA Installation and Service Worker Registration

export const registerServiceWorker = async () => {
  if ('serviceWorker' in navigator) {
    try {
      // Check if sw.js exists before registering
      const response = await fetch('/sw.js', { method: 'HEAD' });
      if (!response.ok) {
        console.log('Service Worker file not found, skipping registration');
        return;
      }

      const registration = await navigator.serviceWorker.register('/sw.js');
      console.log('Service Worker registered successfully:', registration.scope);
      
      // Check for updates periodically
      setInterval(() => {
        registration.update();
      }, 60000); // Check every minute
      
      return registration;
    } catch (error) {
      console.log('Service Worker registration skipped:', error);
      // Don't throw error, just log it - PWA features are optional
    }
  } else {
    console.log('Service Workers not supported in this browser');
  }
};

export const unregisterServiceWorker = async () => {
  if ('serviceWorker' in navigator) {
    const registration = await navigator.serviceWorker.ready;
    await registration.unregister();
    console.log('Service Worker unregistered');
  }
};

// Check if app is installed
export const isAppInstalled = (): boolean => {
  return window.matchMedia('(display-mode: standalone)').matches ||
         (window.navigator as any).standalone === true;
};

// PWA Install Prompt
let deferredPrompt: any = null;

export const setupInstallPrompt = () => {
  window.addEventListener('beforeinstallprompt', (e) => {
    // Prevent the mini-infobar from appearing on mobile
    e.preventDefault();
    // Stash the event so it can be triggered later
    deferredPrompt = e;
    console.log('Install prompt ready');
  });

  window.addEventListener('appinstalled', () => {
    console.log('PWA was installed');
    deferredPrompt = null;
  });
};

export const showInstallPrompt = async (): Promise<boolean> => {
  if (!deferredPrompt) {
    console.log('Install prompt not available');
    return false;
  }

  // Show the install prompt (this triggers the browser's native install)
  deferredPrompt.prompt();

  // Wait for the user to respond to the prompt
  const { outcome } = await deferredPrompt.userChoice;
  console.log(`User response to install prompt: ${outcome}`);

  // Clear the deferred prompt
  deferredPrompt = null;

  return outcome === 'accepted';
};

// Get the deferred prompt for direct access
export const getDeferredPrompt = () => {
  return deferredPrompt;
};

// ⭐ MAIN INSTALLATION FUNCTION - Install app automatically
export const installAppNow = async (): Promise<boolean> => {
  if (!deferredPrompt) {
    console.log('Install prompt not available - app may already be installed or browser does not support PWA');
    return false;
  }

  try {
    // Trigger the browser's install prompt immediately
    console.log('Triggering automatic install...');
    await deferredPrompt.prompt();

    // Wait for the user's response from the browser dialog
    const { outcome } = await deferredPrompt.userChoice;
    console.log(`Install outcome: ${outcome}`);

    // Clear the deferred prompt
    deferredPrompt = null;

    return outcome === 'accepted';
  } catch (error) {
    console.error('Error during installation:', error);
    return false;
  }
};

export const canShowInstallPrompt = (): boolean => {
  return deferredPrompt !== null && !isAppInstalled();
};

// Push notifications
export const requestNotificationPermission = async (): Promise<boolean> => {
  if (!('Notification' in window)) {
    console.log('This browser does not support notifications');
    return false;
  }

  const permission = await Notification.requestPermission();
  return permission === 'granted';
};

export const subscribeToPushNotifications = async (registration: ServiceWorkerRegistration) => {
  try {
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(
        // Replace with your VAPID public key
        'YOUR_VAPID_PUBLIC_KEY_HERE'
      )
    });
    
    console.log('Push subscription:', subscription);
    return subscription;
  } catch (error) {
    console.error('Failed to subscribe to push notifications:', error);
    throw error;
  }
};

// Helper function to convert VAPID key
function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}

// Offline detection
export const setupOfflineDetection = (
  onOnline: () => void,
  onOffline: () => void
) => {
  window.addEventListener('online', onOnline);
  window.addEventListener('offline', onOffline);

  return () => {
    window.removeEventListener('online', onOnline);
    window.removeEventListener('offline', onOffline);
  };
};

export const isOnline = (): boolean => {
  return navigator.onLine;
};
```

---

### 2. Install Modal Component (`/components/InstallPromptModal.tsx`)

```typescript
import { X, Zap, Bell, Smartphone, Download, CheckCircle, AlertCircle } from "lucide-react";
import { motion, AnimatePresence } from "motion/react";
import { Button } from "./ui/button";
import logo from "figma:asset/544b579c188224e698c8dc22f055c8f1ab475a92.png";
import { useState } from "react";
import { installAppNow, canShowInstallPrompt } from "../lib/pwa";
import { toast } from "sonner@2.0.3";

interface InstallPromptModalProps {
  isOpen: boolean;
  onClose: () => void;
  onInstall: () => void;
}

export function InstallPromptModal({ isOpen, onClose, onInstall }: InstallPromptModalProps) {
  const [showConfirmation, setShowConfirmation] = useState(false);

  const handleInstallClick = () => {
    setShowConfirmation(true);
  };

  const handleConfirmInstall = async () => {
    setShowConfirmation(false);
    
    // Close the main modal
    onClose();
    
    // Try to install automatically
    const canInstall = canShowInstallPrompt();
    
    if (canInstall) {
      // Trigger the browser's install prompt immediately
      const installed = await installAppNow();
      
      if (installed) {
        toast.success('App installed successfully! Check your home screen.', {
          duration: 5000,
        });
      } else {
        toast.error('Installation was cancelled or failed. Please try again.', {
          duration: 4000,
        });
      }
    } else {
      // Fallback for browsers that don't support PWA installation
      const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
      const isSafari = /^((?!chrome|android).)*safari/i.test(navigator.userAgent);
      
      if (isIOS || isSafari) {
        toast.info('To install on iOS:\\n\\n1. Tap the Share button\\n2. Tap "Add to Home Screen"\\n3. Tap "Add"', {
          duration: 8000,
        });
      } else {
        toast.info('Look for the install icon (⊕) in your browser\\'s address bar or menu.', {
          duration: 5000,
        });
      }
    }
  };

  const handleCancelInstall = () => {
    setShowConfirmation(false);
  };

  return (
    <AnimatePresence>
      {isOpen && (
        <>
          {/* Backdrop with higher z-index */}
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            onClick={onClose}
            className="fixed inset-0 bg-black/70 backdrop-blur-sm z-[9998]"
          />

          {/* Main Install Modal */}
          <motion.div
            initial={{ opacity: 0, scale: 0.9, y: 20 }}
            animate={{ opacity: 1, scale: 1, y: 0 }}
            exit={{ opacity: 0, scale: 0.9, y: 20 }}
            transition={{ type: "spring", duration: 0.5 }}
            className="fixed inset-0 z-[9999] flex items-center justify-center p-4"
            onClick={(e) => e.stopPropagation()}
          >
            <div className="bg-gradient-to-br from-gray-900 via-gray-800 to-gray-900 rounded-2xl shadow-2xl max-w-md w-full border border-red-500/20 overflow-hidden">
              
              {/* Header with Logo */}
              <div className="relative bg-gradient-to-r from-red-600 to-red-700 p-6 text-center">
                {/* Close Button */}
                <button
                  onClick={onClose}
                  className="absolute top-4 right-4 text-white/80 hover:text-white transition-colors z-10"
                  aria-label="Close"
                >
                  <X className="w-5 h-5" />
                </button>

                {/* Logo */}
                <div className="flex justify-center mb-4">
                  <div className="bg-white rounded-2xl p-3 shadow-lg">
                    <img 
                      src={logo} 
                      alt="Fair Auctioneers Logo" 
                      className="w-16 h-16 object-contain"
                    />
                  </div>
                </div>

                {/* Title */}
                <h2 className="text-white text-2xl mb-2">
                  Install Fair Auctioneers
                </h2>
                <p className="text-red-100 text-sm">
                  Get the best auction experience on your device
                </p>
              </div>

              {/* Content */}
              <div className="p-6">
                {/* Benefits List */}
                <div className="space-y-4 mb-6">
                  <div className="flex items-start gap-3">
                    <div className="flex-shrink-0 w-10 h-10 bg-red-500/20 rounded-lg flex items-center justify-center">
                      <Zap className="w-5 h-5 text-red-400" />
                    </div>
                    <div>
                      <h3 className="text-white mb-1">Lightning Fast</h3>
                      <p className="text-gray-400 text-sm">Instant access and blazing-fast performance</p>
                    </div>
                  </div>

                  <div className="flex items-start gap-3">
                    <div className="flex-shrink-0 w-10 h-10 bg-red-500/20 rounded-lg flex items-center justify-center">
                      <Bell className="w-5 h-5 text-red-400" />
                    </div>
                    <div>
                      <h3 className="text-white mb-1">Real-time Notifications</h3>
                      <p className="text-gray-400 text-sm">Never miss an auction or bid update</p>
                    </div>
                  </div>

                  <div className="flex items-start gap-3">
                    <div className="flex-shrink-0 w-10 h-10 bg-red-500/20 rounded-lg flex items-center justify-center">
                      <Smartphone className="w-5 h-5 text-red-400" />
                    </div>
                    <div>
                      <h3 className="text-white mb-1">One-Tap Access</h3>
                      <p className="text-gray-400 text-sm">Launch directly from your home screen</p>
                    </div>
                  </div>
                </div>

                {/* Action Buttons */}
                <div className="space-y-3">
                  <Button
                    onClick={handleInstallClick}
                    className="w-full bg-gradient-to-r from-red-600 to-red-700 hover:from-red-700 hover:to-red-800 text-white py-6 text-base shadow-lg shadow-red-500/25 hover:shadow-red-500/40 transition-all"
                  >
                    <Download className="w-5 h-5 mr-2" />
                    Install Now
                  </Button>

                  <button
                    onClick={onClose}
                    className="w-full text-gray-400 hover:text-white transition-colors text-sm py-2"
                  >
                    Maybe Later
                  </button>
                </div>
              </div>
            </div>
          </motion.div>

          {/* Confirmation Dialog - Higher z-index than main modal */}
          <AnimatePresence>
            {showConfirmation && (
              <>
                {/* Confirmation Backdrop */}
                <motion.div
                  initial={{ opacity: 0 }}
                  animate={{ opacity: 1 }}
                  exit={{ opacity: 0 }}
                  className="fixed inset-0 bg-black/80 backdrop-blur-md z-[10000]"
                  onClick={handleCancelInstall}
                />

                {/* Confirmation Dialog */}
                <motion.div
                  initial={{ opacity: 0, scale: 0.8, y: 50 }}
                  animate={{ opacity: 1, scale: 1, y: 0 }}
                  exit={{ opacity: 0, scale: 0.8, y: 50 }}
                  transition={{ type: "spring", damping: 20, stiffness: 300 }}
                  className="fixed inset-0 z-[10001] flex items-center justify-center p-4"
                  onClick={(e) => e.stopPropagation()}
                >
                  <div className="bg-gradient-to-br from-gray-900 via-gray-800 to-gray-900 rounded-2xl shadow-2xl max-w-sm w-full border-2 border-red-500/30 overflow-hidden">
                    
                    {/* Icon Header */}
                    <div className="bg-gradient-to-r from-red-600 to-red-700 p-6 text-center">
                      <div className="flex justify-center mb-3">
                        <div className="bg-white/10 backdrop-blur-sm rounded-full p-4">
                          <AlertCircle className="w-12 h-12 text-white" />
                        </div>
                      </div>
                      <h3 className="text-white text-xl font-semibold">
                        Confirm Installation
                      </h3>
                    </div>

                    {/* Content */}
                    <div className="p-6">
                      <p className="text-gray-300 text-center mb-6 leading-relaxed">
                        You're about to install <span className="text-red-400 font-semibold">Fair Auctioneers</span> app on your device. 
                        This will add an icon to your home screen for quick access.
                      </p>

                      {/* Features Preview */}
                      <div className="bg-gray-800/50 rounded-lg p-4 mb-6 space-y-2">
                        <div className="flex items-center gap-2 text-green-400 text-sm">
                          <CheckCircle className="w-4 h-4 flex-shrink-0" />
                          <span>Works offline & loads instantly</span>
                        </div>
                        <div className="flex items-center gap-2 text-green-400 text-sm">
                          <CheckCircle className="w-4 h-4 flex-shrink-0" />
                          <span>Push notifications for auctions</span>
                        </div>
                        <div className="flex items-center gap-2 text-green-400 text-sm">
                          <CheckCircle className="w-4 h-4 flex-shrink-0" />
                          <span>Full-screen app experience</span>
                        </div>
                      </div>

                      {/* Action Buttons */}
                      <div className="space-y-3">
                        <Button
                          onClick={handleConfirmInstall}
                          className="w-full bg-gradient-to-r from-green-600 to-green-700 hover:from-green-700 hover:to-green-800 text-white py-4 text-base font-semibold shadow-lg shadow-green-500/25 hover:shadow-green-500/40 transition-all"
                        >
                          <CheckCircle className="w-5 h-5 mr-2" />
                          Yes, Install App
                        </Button>

                        <button
                          onClick={handleCancelInstall}
                          className="w-full bg-gray-700/50 hover:bg-gray-700 text-gray-300 hover:text-white py-4 rounded-lg text-base font-medium transition-all"
                        >
                          Cancel
                        </button>
                      </div>
                    </div>
                  </div>
                </motion.div>
              </>
            )}
          </AnimatePresence>
        </>
      )}
    </AnimatePresence>
  );
}
```

---

### 3. Navbar Implementation (Trigger Point)

```typescript
// In /components/Navbar.tsx

// Import the PWA functions
import { canShowInstallPrompt, isAppInstalled, installAppNow } from '../lib/pwa';
import { InstallPromptModal } from './InstallPromptModal';

// Inside your Navbar component:
const [showInstallButton, setShowInstallButton] = useState(true);
const [showInstallModal, setShowInstallModal] = useState(false);

// Check if install button should be shown
useEffect(() => {
  const checkInstallPrompt = () => {
    const canShow = canShowInstallPrompt();
    const installed = isAppInstalled();
    
    // Hide button only if already installed
    if (installed) {
      setShowInstallButton(false);
    } else {
      setShowInstallButton(true);
    }
  };

  // Check immediately
  checkInstallPrompt();

  // Listen for the beforeinstallprompt event
  const handleBeforeInstallPrompt = () => {
    console.log('beforeinstallprompt event fired!');
    setTimeout(checkInstallPrompt, 100);
  };

  window.addEventListener('beforeinstallprompt', handleBeforeInstallPrompt);

  // Check periodically in case the prompt becomes available
  const interval = setInterval(checkInstallPrompt, 2000);

  return () => {
    clearInterval(interval);
    window.removeEventListener('beforeinstallprompt', handleBeforeInstallPrompt);
  };
}, []);

const handleInstallClick = async () => {
  // Always show custom modal first
  setShowInstallModal(true);
};

const handleInstallConfirm = async () => {
  setShowInstallModal(false);
  
  // Check if the browser supports PWA installation
  const canShow = canShowInstallPrompt();
  
  if (canShow) {
    const accepted = await installAppNow();
    if (accepted) {
      setShowInstallButton(false);
    }
  } else {
    // Fallback for browsers that don't support beforeinstallprompt
    const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
    const isSafari = /^((?!chrome|android).)*safari/i.test(navigator.userAgent);
    
    if (isIOS || isSafari) {
      alert('To install this app on iOS:\\n\\n1. Tap the Share button (square with arrow)\\n2. Scroll down and tap "Add to Home Screen"\\n3. Tap "Add" to confirm\\n\\nThe app icon will appear on your home screen!');
    } else {
      alert('To install this app:\\n\\n• Chrome/Edge: Look for the install icon (⊕) in the address bar\\n• Or use browser menu → "Install Fair Auctioneers"\\n\\nThis will add the app to your device!');
    }
  }
};

// In your JSX:
{showInstallButton && releaseConfig.features.pwa && (
  <Button 
    className="bg-gradient-to-r from-red-600 to-red-700 hover:from-red-700 hover:to-red-800" 
    onClick={handleInstallClick}
  >
    <Download className="w-5 h-5 mr-2" />
    Install our App
  </Button>
)}

<InstallPromptModal
  isOpen={showInstallModal}
  onClose={() => setShowInstallModal(false)}
  onInstall={handleInstallConfirm}
/>
```

---

## 🎨 Z-Index Layer System

```
Layer 1: Main Page Content              → z-index: default (0-100)
Layer 2: Main Modal Backdrop             → z-index: 9998
Layer 3: Main Modal                      → z-index: 9999
Layer 4: Confirmation Backdrop (darker)  → z-index: 10000
Layer 5: Confirmation Dialog (top)       → z-index: 10001
```

---

## 🚀 Installation Flow

```
┌─────────────────────────────────────────┐
│ 1. User clicks "Install our App"       │
│    (Navbar button)                      │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 2. Main Install Modal Opens             │
│    - Shows company logo                 │
│    - Lists 3 benefits                   │
│    - "Install Now" button               │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 3. User clicks "Install Now"            │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 4. Confirmation Dialog Opens            │
│    - Higher z-index (on top)            │
│    - "Yes, Install App" button          │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 5. User clicks "Yes, Install App"       │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 6. installAppNow() function executes    │
│    - Triggers browser install prompt    │
│    - Shows browser's native dialog      │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 7. Browser Shows Install Dialog         │
│    (ONE TIME - Automatic)               │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 8. User accepts in browser dialog       │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ 9. App Installs to Home Screen          │
│    - With your company logo             │
│    - Toast notification shows success   │
└─────────────────────────────────────────┘
```

---

## ⚙️ Manifest.json Configuration

Make sure your `/public/manifest.json` has:

```json
{
  "name": "Fair Auctioneers",
  "short_name": "Fair Auctions",
  "description": "Sri Lanka's Premier Vehicle Auction Platform",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#dc2626",
  "icons": [
    {
      "src": "/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

**Important:** Replace `/icon-192x192.png` and `/icon-512x512.png` with your actual company logo files!

---

## 🌐 Browser Support

| Browser | Support | Notes |
|---------|---------|-------|
| Chrome (Android) | ✅ Full | Best experience |
| Edge (Android) | ✅ Full | Best experience |
| Firefox (Android) | ✅ Full | Best experience |
| Safari (iOS) | ⚠️ Partial | Manual install only (shows instructions) |
| Chrome (Desktop) | ✅ Full | Desktop PWA support |
| Firefox (Desktop) | ⚠️ Limited | Limited PWA support |

---

## 🔍 Key Functions Explained

### `installAppNow()`
- **Purpose:** Automatically trigger browser's PWA install prompt
- **Returns:** `true` if installed, `false` if cancelled/failed
- **Limitation:** Browser security requires user to interact with browser dialog

### `canShowInstallPrompt()`
- **Purpose:** Check if PWA can be installed
- **Returns:** `true` if `beforeinstallprompt` event captured and app not already installed

### `isAppInstalled()`
- **Purpose:** Check if app is already installed
- **Returns:** `true` if running in standalone mode (installed)

---

## 🎯 Important Notes

### ❗ Browser Security Restrictions:
1. **Cannot bypass browser dialog:** Browsers require users to confirm installation for security
2. **HTTPS required:** PWA installation only works on HTTPS domains (or localhost)
3. **Service Worker required:** Must have a valid service worker registered
4. **Manifest required:** Must have a valid `manifest.json` file

### ✅ What IS Automatic:
- Opening the browser's install dialog immediately after confirmation
- Using your company logo from manifest.json
- Adding to home screen after user accepts browser dialog

### ❌ What is NOT Automatic:
- Skipping the browser's native install confirmation (security restriction)
- Installing without user interaction (security restriction)

---

## 🐛 Troubleshooting

### Install button doesn't appear:
1. Check console for `beforeinstallprompt` event
2. Verify manifest.json is valid
3. Ensure HTTPS is enabled
4. Check if app is already installed

### Browser dialog doesn't show:
1. Verify `deferredPrompt` is captured
2. Check console logs for errors
3. Test on supported browser (Chrome/Edge Android)

### Logo doesn't appear after install:
1. Check manifest.json icon paths
2. Ensure icon files exist in `/public`
3. Clear browser cache and reinstall

---

## 📊 Testing Checklist

- [ ] Test on Chrome Android
- [ ] Test on Safari iOS (manual install instructions)
- [ ] Verify company logo appears in modal
- [ ] Verify logo appears on home screen after install
- [ ] Test toast notifications (success/error)
- [ ] Test "Maybe Later" and "Cancel" buttons
- [ ] Verify z-index layering (confirmation on top)
- [ ] Test already-installed detection
- [ ] Verify install button disappears after install

---

## 🎉 Summary

This PWA installation system provides:
- **Beautiful UI** with company branding
- **Two-step confirmation** to prevent accidents
- **Automatic browser trigger** (as automatic as browsers allow)
- **Smart fallbacks** for unsupported browsers
- **Toast notifications** for user feedback
- **Your company logo** on installed app icon

The system is production-ready and follows best practices for PWA installation!

---

**Created for:** Fair Auctioneers (Pvt) Ltd  
**Version:** 1.0.0  
**Last Updated:** January 2026
