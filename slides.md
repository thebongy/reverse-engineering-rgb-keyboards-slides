---
theme: seriph
background: https://images.unsplash.com/photo-1626958390943-a70309376444?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2070&q=80
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Reverse Engineering RGB Keyboard Backlights
  Talk at Nullcon Goa 2023
drawings:
  persist: false
transition: slide-left
title: Reverse Engineering RGB Keyboard Backlights
mdc: true
hideInToc: true
---

# Reverse Engineering RGB Keyboard Backlights

Rishit Bansal

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
layout: image-right
image: /rishit.jpeg
hideInToc: true
---

# 👤 Hi, I'm Rishit Bansal!

- From Bangalore, India
- Software Developer @ Dyte (https://dyte.io)
- I play CTFs with csictf, dytesec

<br>
<br>


<img src="/linkedin.png" class="w-4 mr-2 inline" />
<a href="https://www.linkedin.com/in/bansal-rishit/">https://www.linkedin.com/in/bansal-rishit/</a>

<img src="/mail.webp" class="w-4 mr-2 inline" />
<a href="mailto:rishit.bansal0@gmail.com">rishit.bansal0@gmail.com</a>

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<!--
Here is another comment.
-->

---
layout: center
hideInToc: true
---

# 🎯 What will we cover in this talk?

- Chapter 1: Reverse Engineer a Windows Service which interacts with keyboard backlight firmware
- Chapter 2: Understand the system protocols involved in interacting with device firmware
- Chapter 3: Re-implement the same functionality on the Linux Kernel
- Bonus: Discover new functionality possible in hardware!

<br />
<br />

<!-- ### Chapters
<Toc maxDepth="1"></Toc> -->

---
level: 1
layout: center
---

## Chapter 1
# Reverse Engineering the HP Light Studio Application

---
level: 2
layout: image-right
image: /laptop.jpg
---

# The Laptop

- Laptop: HP Omen 15
- Four zones of configurable backlights
- Can set colors at a really fast rate to simulate animations.
- Hardware Key to toggle the backlight on and off.


---
level: 2
layout: default
---

# HP Omen Light Studio Application

<div class="flex justify-center items-center mt-16 space-x-8">

<img src="/lightStudio.jpeg" class="w-120" />


<ul>
<li>Windows Desktop Application</li>
<li>Allows you to choose between template/custom animations</li>
<li>Closed source, 0 documentation</li>
<li>Animations only work when Windows boots, otherwise just static colors</li>
</ul>

</div>

---
level: 2
layout: default
---

# Where is the service?

<div class="flex justify-center items-center mt-16 space-x-8">
<div class="flex flex-col items-center space-y-8">
<img src="/taskmanager.png" class="w-120" />
<div>Omen Gaming Hub on Task Manager <br /> <kbd>C:\Program Files\WindowsApps\AD2F1837.OMENLightStudio_1.0.37.0_x64__v10z8vjag6ke6</kbd></div>
</div>
<div class="flex flex-col items-center space-y-8">
<img src="/taskmanager2.png" class="w-120" />
<div>
Light Studio Service on Task Manager <br />
<kbd>C:\Program Files\HP\LightStudioHelper</kbd>
</div>

</div>

</div>


---
level: 2
layout: image
image: /helperfolder.png
---


---
level: 2
layout: center
---

# Decompiling the Executable and DLL Files <br /> (Ilspy Demo)

---
level: 2
layout: full
---

```cs {6,7,8,12,13,14,15,16} {lines:true}
private static int Execute(int command, int commandType, int inputDataSize, byte[] inputData, out byte[] returnData)
{
	returnData = new byte[0];
	try
	{
		ManagementObject val = new ManagementObject(
      "root\\wmi", "hpqBIntM.InstanceName='ACPI\\PNP0C14\\0_0'", (ObjectGetOptions)null
      );
		ManagementObject val2 = (ManagementObject)new ManagementClass("root\\wmi:hpqBDataIn");
		ManagementBaseObject methodParameters = val.GetMethodParameters("hpqBIOSInt128");
		ManagementBaseObject val3 = (ManagementBaseObject)new ManagementClass("root\\wmi:hpqBDataOut128");
		((ManagementBaseObject)val2).set_Item("Sign", (object)Sign);
		((ManagementBaseObject)val2).set_Item("Command", (object)command);
		((ManagementBaseObject)val2).set_Item("CommandType", (object)commandType);
		((ManagementBaseObject)val2).set_Item("Size", (object)inputDataSize);
		((ManagementBaseObject)val2).set_Item("hpqBData", (object)inputData);
		methodParameters.set_Item("InData", (object)val2);
		InvokeMethodOptions val4 = new InvokeMethodOptions();
		((ManagementOptions)val4).set_Timeout(TimeSpan.MaxValue);
		InvokeMethodOptions val5 = val4;
		object obj = val.InvokeMethod("hpqBIOSInt128", methodParameters, val5).get_Item("OutData");
		val3 = (ManagementBaseObject)((obj is ManagementBaseObject) ? obj : null);
		returnData = val3.get_Item("Data") as byte[];
		return Convert.ToInt32(val3.get_Item("rwReturnCode"));
	}
	catch (Exception ex)
	{
		Console.WriteLine("OMEN Four zone lighting - WmiCommand.Execute occurs exception: " + ex);
		return -1;
	}
}
```
---
level: 2
layout: default
---

# Chapter 1 Summary

- Used ILSpy to decompile a .NET Windows Service, identified how the Light Studio Helper Service Works
- Mysterious References to other protocols and Windows APIs
  - Windows C# API: `ManagementObject`
  - References to `root\\wmi`
  - `ACPI\\PNP0C14\\0_0`
- "Command" parameter is passed `131081`
  - Type `1`: Checks if lighting is supported
  - Type `2`: Getting the colors of each zone on the keyboard
  - Type `3`: Setting the colors of each zone on the keyboard
  - Type `4`: Checks if backlight is on or off
  - Type `5`: Sets brightness for each of the 4 zones
---
level: 1
layout: center
---

# Chapter 2: Understanding the System Protocols!

---
level: 1
layout: default
image: /8086.png
---

### Flow #1: How does a keypress toggle the backlight?

<div class="flex justify-center items-center  space-x-8">

<img src="/stage1.png" style="width:630px;margin-top:10px;" />
<div>

</div>

</div>

---
level: 1
layout: default
image: /8086.png
---

### Flow #1: What are the disadvantages of this approach?
<div class="flex items-center justify-center space-x-8 mt-4">

<img src="/ivt.png" style="width:300px;margin-top:10px;" />
<div>

- No standardization on interrupt numbers, vectors
- I/O Device API handlers were hardcoded in Bios Firmware
- OS/Kernel space software has no direct I/O accees, functionality hardcoded in Bios
  - Needed in modern systems (read temp sensors, power management, etc.)

</div>

</div>

---
level: 1
layout: default
---

### Flow #2: How does the keyboard toggle the backlight? (ACPI)

<div class="flex justify-center">

<img src="/stage2.png" style="width:600px;margin-top:10px;" />

</div>

---
level: 1
layout: default
---
<div class="flex justify-center items-center flex-col">
<div>

# ACPI Architecture

</div>
<img src="/acpi.png" class="h-100" />
</div>

---
level: 1
layout: default
---
# Reading/Decompiling AML code on Linux

- AML Code is loaded into main memory on boot by firmware
  - Linux ACPI Driver mounts this at `/sys/firmware/acpi/tables/DSDT`
- `iasl -d <dsdt_dump>` CLI tool can be used to decompile AML bytecode

```c {3,5,13,14,15,16} {lines:true}
  Scope (_SB)
    {
        Device (ACAD)
        {
            Name (_HID, "ACPI0003" /* Power Source Device */)  // _HID: Hardware ID
            Name (_PCL, Package (0x01)  // _PCL: Power Consumer List
            {
                _SB
            })
            Name (XX00, Buffer (0x03) {})
            Name (ACSB, One)
            Name (ACDC, 0xFF)
            Method (_STA, 0, NotSerialized)  // _STA: Status
            {
                Return (0x0F)
            }
...
```

---
level: 1
layout: center
---
## But all this is still on the kernel, how do I toggle the backlight from userspace?


---
level: 1
layout: default
---
# WMI (Windows Management Instrumentation)

- A protocol to help sysadmins manage distributed network of windows machines.
  - Execute management scripts/retreieve information from machines.
- Runs a simple server which accepts requests from clients, and executes them on "WMI Providers"
  - Providers expose classes and methods, each class has a unique "GUID"
- Simply put, just a way to do RPC between applications on windows machines.

<br />
<br />
<div class="flex items-center justify-center">
<img src="/wmi.png" class = "w-200" />
</div>

---
level: 1
layout: default
---
# WMI Explorer

<div class="flex items-center justify-center">
<img src="/wmiexplorer.png" class = "w-180" />
</div>

---
level: 1
layout: default
---
# How it all links together: WMI-ACPI!

- Microsoft's proprietary extenstion to the ACPI specification
- Allows you to expose ACPI Methods, as WMI Methods on Windows.
- Developer must create a custom ACPI device with ID `PNP0C14` in AML.
  - Must have a field called `_WDG`
  - `_WDG` stores metadata for links between WMI Class GUIDs and ACPI Functions (Wmxx)

```c {} {lines:true}

Device (WMID)
        {
            Name (_HID, "PNP0C14" /* Windows Management Instrumentation Device */)  // _HID: Hardware ID
            Name (_WDG, Buffer (0x0118)
            {
                /* 0000 */  0x34, 0xF0, 0xB7, 0x5F, 0x63, 0x2C, 0xE9, 0x45,  // 4.._c,.E
                /* 0008 */  0xBE, 0x91, 0x3D, 0x44, 0xE2, 0xC7, 0x07, 0xE4,  // ..=D....
                /* 0010 */  0x41, 0x41, 0x01, 0x02, 0x79, 0x42, 0xF2, 0x95,  // AA..yB..
                /* 0018 */  0x7B, 0x4D, 0x34, 0x43, 0x93, 0x87, 0xAC, 0xCD,  // {M4C....
                /* 0020 */  0xC6, 0x7E, 0xF6, 0x1C, 0x80, 0x00, 0x01, 0x08,  // .~......
```

---
level: 1
layout: default
---
# Using wmidump to read WDG buffers

- `$ wmidump < file_with_wdg_buffer`
- Extracts out the WMI method GUIDs and ACPI function mappings

```c
5FB7F034-2C63-45E9-BE91-3D44E2C707E4:
	object_id: AA
	instance_count: 1
	flags: 0x2 ACPI_WMI_METHOD 
95F24279-4D7B-4334-9387-ACCDC67EF61C:
	notify_id: 0x80
	instance_count: 1
	flags: 0x8 ACPI_WMI_EVENT 
....
```
- WMAA Method in AML Code:
```c
Method (WMAA, 3, Serialized)
{
    Acquire (MUTZ, 0xFFFF)
    Local0 = HWMC (Arg1, Arg2)
    Release (MUTZ)
    Return (Local0)
}
```
---
level: 1
layout: center
---
#### WMI-ACPI Flow for Omen Light Studio Application

<div class="flex items-center justify-center mt-2">
<img src= "/wmiacpi.png" class="h-125" />
</div>
---
level: 2
layout: default
---

# Chapter 2 Summary

- Learned about the innter workings of two protocols, ACPI and WMI.
- ACPI specifies "AML" code loaded from BIOS into main memory.
  - OS (Eg: Linux) can read and execute this to interact with I/O devices.
- WMI provides a way for user-space communication between services on Windows.
- WMI-ACPI exposes ACPI methods as WMI Methods

<br /><br /><br />

## Plan: Implement a kernel driver to interface WMI/ACPI on Linux!
---
level: 1
layout: center
---
# Chapter 3: Developing WMI Drivers on the Linux Kernel

---
level: 1
layout: default
---
# acpi.h in the Linux Kernel

- Implementation of the ACPI specification / AML interpretor
- `acpi_boot_init()` is called on boot to parse AML from ACPI tables in system-memory
- Provides helper functions to invoke WMI-ACPI methods using their GUID directly from a kernel driver:
```c
extern acpi_status wmi_evaluate_method(const char *guid, u8 instance,
					u32 method_id,
					const struct acpi_buffer *in,
					struct acpi_buffer *out);
```

---
level: 1
layout: default
---
# Interfacing kernel APIs from userspace

- `sysfs` allows you to create custom files in the /sys/ to represent device driver APIs
  - You can write/read to these files from userspace, to trigger handlers in the kernel
- In our case, /sys/class/leds is most relevant to represent the keyboard backlight device
  - Has special handlers we have to implement for brightness control
  - Added a custom file called `zone_colors` to represent the RGB backlights

```c
static DEVICE_ATTR_RW(zone_colors);
static struct attribute *omen_kbd_led_attrs[] = {
	&dev_attr_zone_colors.attr,
	NULL,
};
ATTRIBUTE_GROUPS(omen_kbd_led);

static struct led_classdev omen_kbd_led = {
	.name = "hp_omen::kbd_backlight",
	.brightness_set = set_omen_backlight_brightness,
	.brightness_get = get_omen_backlight_brightness,
	.max_brightness = 1,
	.groups = omen_kbd_led_groups,
};
```

---
level: 1
layout: center
---
# What the new device driver file tree looks like


```bash
rishit@OMEN-laptop:~/Documents/kernels/staging$ ls /sys/class/leds/hp_omen\:\:kbd_backlight/
brightness  max_brightness  subsystem  uevent
device      power           trigger    zone_colors

```

<br />
<br />
<br />
- We need to write handlers to read/write `brightness` and `zone_colors`

---
level: 1
layout: center
---

```c

#define HPWMI_READ_ZONE 0x02
#define HPWMI_WRITE_ZONE 0x03
#define OMEN_ZONE_COLOR_LEN 0x0c // 12 bytes (3 components (R,G,B) * 4 zones)
#define OMEN_ZONE_COLOR_OFFSET 0x19 // 25
#define HPWMI_KB 0x20009 // 131081

static ssize_t zone_colors_store(struct device *dev, struct device_attribute *attr, const char *buf, size_t count)
{
	u8 val[128];
	int ret;

	ret = hp_wmi_perform_query(HPWMI_READ_ZONE, HPWMI_KB, &val, zero_if_sup(val), sizeof(val));

	if (ret)
		return ret;

	if (count != OMEN_ZONE_COLOR_LEN)
		return -1;

	memcpy(&val[OMEN_ZONE_COLOR_OFFSET], buf, count);

	ret = hp_wmi_perform_query(HPWMI_WRITE_ZONE, HPWMI_KB, &val, sizeof(val), 0);

	if (ret)
		return ret;

	return OMEN_ZONE_COLOR_LEN;
}

```
---
level: 1
layout: default
---
### Handlers for brightness control
<br />

```c

#define HPWMI_WRITE_BRIGHTNESS 0x05 

#define HPWMI_KB 0x20009 // 131081

static void set_omen_backlight_brightness(struct led_classdev *cdev, enum led_brightness value)
{
	char buffer[4] = { (value == LED_OFF) ? 0x64 : 0xe4, 0, 0, 0 };

	hp_wmi_perform_query(HPWMI_WRITE_BRIGHTNESS, HPWMI_KB, &buffer,
				       sizeof(buffer), 0);
}

```

- **Bonus**: Even though this LED only supports ON and OFF state, we can *simulate* brightness using a trick
- We can use the previous zone_colors_store/read methods and "scale" the RGB components by the brightness multipler
- $R_{eff}(zone) = R(zone) * (brightness/100)$
---
layout: fact
---

## Demo
---
layout: fact
---

## Thank you!

<br />
<br />

### Questions?
---