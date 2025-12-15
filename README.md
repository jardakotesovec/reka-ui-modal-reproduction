# Reka-UI FocusScope + TinyMCE Focus Trap Issue

This is a minimal reproduction of an issue where TinyMCE dialogs cannot receive focus when opened inside a reka-ui Dialog (modal).

## Setup

```bash
npm install
npm run dev
```

## Steps to Reproduce

1. Open the app in your browser
2. Click "Open Modal" button
3. In the TinyMCE editor inside the modal, click the "Link" button in the toolbar
4. Try to type in the URL input field

## Expected Behavior

The TinyMCE link dialog inputs should be focusable and allow typing.

## Actual Behavior

Focus is immediately returned to the modal container. The TinyMCE dialog inputs appear disabled - you can click them but cannot type.

## Environment

- reka-ui: 2.x
- Vue: 3.x
- TinyMCE: 7.x
