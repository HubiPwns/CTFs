# Binwalk Writeup

This challenge required analyzing a file with **binwalk** to extract hidden data and recover the flag.

---

## Step 1: Initial analysis
I started with a simple scan:

```binwalk PurpleThing.jpeg```

![[Zrzut ekranu 2025-09-17 181202 2.png]]

---
## Step 2: First extraction attempt

I tried the standard extraction:

```binwalk -e PurpleThing.jpeg```

![[Zrzut ekranu 2025-09-17 181626 1.png]]

---
## Step 3: Forcing extraction of PNG files
To get everything out, I used the following command:

```binwalk -e -D 'png:png' PurpleThing.jpeg```

![[Zrzut ekranu 2025-09-17 181843.png]]

This successfully extracted additional files.

---

## Step 4: Finding the flag
After inspecting the extracted files, I found the flag:

🎯 **Flag:**  
`ABCTF{b1nw4lk_is_us3ful}`
