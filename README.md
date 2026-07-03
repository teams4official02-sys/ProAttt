# ProAttt Firebase Connect App

A premium single-page HTML app for connecting to a Firebase Realtime Database URL without an API key.

## Run locally

From this repository folder, start a simple static server:

```bash
cd /workspace/ProAttt
python3 -m http.server 4173 --bind 127.0.0.1
```

Then open:

```text
http://127.0.0.1:4173/index.html
```

## Fix: `fatal: not a git repository`

If you see this error:

```text
fatal: not a git repository (or any parent up to mount point /)
Stopping at filesystem boundary (GIT_DISCOVERY_ACROSS_FILESYSTEM not set).
bash: cd: null directory
```

You are running Git from the wrong folder, or your script/editor is trying to `cd` into an empty or `null` path.

Use this command first:

```bash
cd /workspace/ProAttt
```

Then confirm Git can see the repository:

```bash
git status
```

If you cloned the project somewhere else, replace `/workspace/ProAttt` with your actual project path.

## Firebase URL format

Use your Firebase Realtime Database URL, for example:

```text
https://your-project-default-rtdb.firebaseio.com
```

Do not use a Firebase Hosting URL or Firestore URL on the connect screen.


## App flow

1. Login screen par Firebase Realtime Database URL enter karein.
2. Connect hone ke baad app Home screen par redirect hota hai.
3. Home screen par `S4 AUTO PANEL` title ke niche two cards milte hain:
   - `ONLINE`: Firebase data me `status: true` ya `stutas: true` wale clients ko card form me show karta hai.
   - `CHACKED`: Abhi coming soon hai; future me feature add hoga.
4. Online client card par click karne se app us device se connect hota hai aur `S4 AUTO SYSTEM` screen open karta hai.
5. Connected device screen par target number, SIM slot, aur massage enter karke `Send Massage` click karne se pehle primary path `clients/<clientId>/webhookEvent/sendSms` par `{ from, isSended: false, message, to }` format me save hota hai; agar primary fail ho to fallback path `clients/<clientId>/sendSms` par same data save hota hai.
6. `Show Device Massages` se connected device ke available `massages`, `messages`, `smsCommands`, aur `sentMessages` nodes read hote hain.

7. Home screen ka `TELEGRAM` card channel username save karta hai. Static HTML app direct Telegram channel read nahi kar sakta, isliye channel messages ko Firebase path `telegramChannels/<channel>/messages` me mirror karein; app connected client open hone par 2 second interval me messages parse karke SMS command bhejta hai.
8. Telegram message parser `To`, `Number`, `Phone`, `Target` se number aur `Massage`, `Message`, `Body`, `MSG`, `SMS`, `Text`, `Content` se SMS body extract karta hai.

9. Device massages newest-to-oldest order me show hote hain aur cards screenshot-style yellow/white layout me render hote hain.
10. Telegram channel polling har 2 second me hoti hai; new channel massage detect hote hi 2 second ka popup show hota hai, aur agar client connected hai to parsed SMS usi connected client se send hota hai.

11. Message cards ab lower height/smaller font ke saath shadow me visible hote hain, aur connected device messages screen open hone par har 0.5 second me auto-refresh hoti hai.
12. `Show Device Massages` button explicitly opens the messages screen and starts the 0.5 second refresh loop.
13. Message refresh overlapping requests ko lock/queue karta hai, latest messages ko timestamp/Firebase push-id ke basis par top-to-bottom order me rakhta hai, aur nested message nodes bhi collect karta hai.
14. Online devices ab list format me device id, device name, battery aur ONLINE status ke saath show hote hain. Agar `clients/<clientId>` ke andar kisi bhi nested node me `status/stutas: true` ho to app poore `clients/<clientId>` ko selectable device maanta hai, metadata nested nodes se bhi read karta hai, aur messages sirf selected client ke own paths se load karta hai taaki dusre client ke messages mix na hon.
