# APEX Thermal Print & Cash Drawer Utility

A lightweight JavaScript and PL/SQL utility designed for Oracle APEX applications to handle direct client-side thermal receipt printing (ESC/POS) and cash drawer triggering via the WebUSB API without requiring intermediate print servers.

## Features
* **Direct Thermal Printing:** Sends raw ESC/POS command sequences directly to USB thermal receipt printers from the browser.
* **Cash Drawer Integration:** Triggers cash drawer kicks seamlessly via JavaScript/WebUSB.
* **Plug & Play:** Easy integration into Oracle APEX pages using standard Execute JavaScript Code dynamic actions.

## Prerequisites
* Oracle APEX 21.x or higher.
* Modern web browser supporting the WebUSB API (Google Chrome, Microsoft Edge).
* USB Thermal Receipt Printer (EPSON, POS-58/80, etc.).

## Installation & Usage

1. **Setup Page Item:** Create a hidden page item in your APEX page (e.g., `P2_PAY_LOAD`) to hold the print payload data.
2. **Add Dynamic Action:** Create a Dynamic Action on your print button (e.g., `PRINT_RECEIPT_BTN`) with the following JavaScript code:

```javascript
// Example WebUSB Thermal Print Trigger
async function printReceipt() {
    try {
        const payload = apex.item("P2_PAY_LOAD").getValue();
        const encoder = new TextEncoder();
        const data = encoder.encode(payload);

        // Request USB Device connection
        const device = await navigator.usb.requestDevice({ filters: [{ vendorId: 0x0483 }] }); // Replace with your printer vendor ID
        await device.open();
        await device.selectConfiguration(1);
        await device.claimInterface(0);

        // Send ESC/POS payload to endpoint (Endpoint 1 out typically)
        await device.transferOut(1, data);
        console.log("Receipt printed successfully!");
    } catch (error) {
        console.error("Printing failed: ", error);
    }
}

printReceipt();
