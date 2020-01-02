# Acer Aspire VX5-576G Hackintosh
__My Specs__

| Specs | Details |
|------------|-------------------------------|
| Model | Acer Aspire VX15 (VX5-591G) |
| OS | macOS Catalina 10.15.2 |
| CPU | Intel(R) Core(TM) i5 7300HQ |
| RAM | 16 GB DDR4 2400MHz |
| iGPU | Intel HD Graphics 630 |
| dGPU | NVIDIA GeForce GTX 1050M |
| Touchpad | ELAN 0501 |
| Wireless | Intel® Dual Band Wireless-AC 7265 |
| Audio | ALC255 |

__Tested and working__

- Intel HD Graphics 630
- USB
- Webcam
- LAN
- Bluetooth (Can't off)
- Screen Brightness
- Battery status
- Sleep/Wake
- Audio
- TouchPad with gestures
- Keyboard with backlight (Some function keys not work)

__Not tested__

- USB C
- HDMI

__Not working__

- NVIDIA GeForce GTX 1050M
- SD Card reader
- WiFi (Replace another wifi card that supports hackintosh)

__There are also a few other bugs that I can't fix__

## Guide

__BIOS Settings (Version 1.08)__

- Set Supervisor Password
- Disable Password on Boot
- Disable Secure Boot
- Set touchpad: Advance

__Patch your DSDT__

All the necessary kext already have you just need to patch your DSDT.  
RehabMan guide: https://www.tonymacx86.com/threads/guide-patching-laptop-dsdt-ssdts.152573/  
After fixing all the errors and warning, you need to apply the following basic patches:  

- HPET Fix 
- SMBUS Fix 
- IRQ Fix 
- RTC Fix 
- OS Check Fix 
- 7-series/8-series USB 
- USB3_PRW 0x6D Skylake (instant wake) 
- Audio Layout 30

__Fix Touchpad__

Patch VoodooI2C for your DSDT.  
In the name column write `VoodooI2C` and put `http://raw.github.com/alexandred/VoodooI2C-Patches/master` as the URL.  
Patch `VoodooI2CGPIOGPIO Controller Enable`  

__Disabling discrete graphics and fix Sleep/Wake__

You need to turn off your NVIDIA card for it to work.  
Patch your DSDT.  
On the top of DSDT:  
```
External (_SB_.PCI0.PEG0.PEGP._PS3, MethodObj) 
External (_SB_.PCI0.PEG0.PEGP._PS0, MethodObj) 
External (_SB_.PCI0.PEG0.PEGP._OFF, MethodObj) 
External (_SB_.PCI0.PEG0.PEGP._ON, MethodObj) 
External (_SB_.PCI0.PEG0.PEGP.SGOF, MethodObj) 
External (_SB_.PCI0.PEG0.PEGP.SGON, MethodObj) 
```
Find _WAK Method and add these lines:  
```
Method (M_ON, 0, NotSerialized)
    {
        If (CondRefOf (\_SB_.PCI0.PEG0.PEGP._ON))
        {
            \_SB_.PCI0.PEG0.PEGP._ON()
        }
        If (CondRefOf (\_SB_.PCI0.PEG0.PEGP._PS0))
        {
            \_SB_.PCI0.PEG0.PEGP._PS0()
        }
        If (CondRefOf (\_SB_.PCI0.PEG0.PEGP.SGON))
        {
            \_SB_.PCI0.PEG0.PEGP.SGON()
        }
    }
```

```
Method (M_OF, 0, NotSerialized)
    {
        If (CondRefOf (\_SB_.PCI0.PEG0.PEGP._OFF))
        {
            \_SB_.PCI0.PEG0.PEGP._OFF()
        }
        If (CondRefOf (\_SB_.PCI0.PEG0.PEGP._PS3))
        {
            \_SB_.PCI0.PEG0.PEGP._PS3()
        }
        If (CondRefOf (\_SB_.PCI0.PEG0.PEGP.SGOF))
        {
            \_SB_.PCI0.PEG0.PEGP.SGOF()
        }
    }
```
Then add `M_OF()` Method to _INI and _WAK Methods  
In order to get sleep to work:  
Find _PTS Method and add `M_ON()`  

__Fix brightness key__
```
into method label _Q11 replace_content
begin
// Brightness Down\n
    Notify(\_SB.PCI0.LPCB.PS2K, 0x0405)\n
end;
into method label _Q12 replace_content
begin
// Brightness Up\n
    Notify(\_SB.PCI0.LPCB.PS2K, 0x0406)\n
end;
```

*__Enjoy__*