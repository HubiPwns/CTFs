# Binwalk [easy]

This challenge required analyzing a file with **binwalk** to extract hidden data and recover the flag.

---

## Step 1: Initial analysis
I started with a simple scan:

```binwalk PurpleThing.jpeg```

<img width="1072" height="348" alt="Zrzut ekranu 2025-09-17 181202" src="https://github.com/user-attachments/assets/6401f2fe-d5a6-48f9-b279-f5f9ef8a9fdb" />

---
## Step 2: First extraction attempt

I tried the standard extraction, but it didn’t extract all the files:

```binwalk -e PurpleThing.jpeg```

<img width="1225" height="232" alt="Zrzut ekranu 2025-09-17 181626" src="https://github.com/user-attachments/assets/d90942c7-5909-4110-84cc-13c7b452a62f" />

---
## Step 3: Forcing extraction of PNG files
To get everything out, I used the following command:

```binwalk -e -D 'png:png' PurpleThing.jpeg```

<img width="1060" height="247" alt="Zrzut ekranu 2025-09-17 181843" src="https://github.com/user-attachments/assets/9b45d8eb-e4c5-4965-ad40-6628224d9302" />

This successfully extracted additional files.

---

## Step 4: Finding the flag
After inspecting the extracted files, I found the flag:

🎯 **Flag:**  
`ABCTF{b1nw4lk_is_us3ful}`
