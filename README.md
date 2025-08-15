# PrepSmart
# Clone your repo
git clone https://github.com/jayeshupadhyaydec-prog/PrepSmart.git

# Copy my generated files into this folder
cd PrepSmart

# Add files to git
git add .
git commit -m "Added full PrepSmart app code"

# Push to GitHub
git push origin main
PrepSmart/
│
├─ package.json
├─ README.md
├─ .gitignore
├─ .env.example
│
├─ packages/
│  └─ shared/
│     ├─ package.json
│     └─ src/api.ts
│
└─ apps/
   ├─ backend/
   │  ├─ package.json
   │  ├─ src/
   │  │  ├─ index.ts
   │  │  ├─ routes/
   │  │  │  ├─ ai.ts
   │  │  │  └─ redeem.ts
   │  └─ tsconfig.json
   │
   ├─ web/                # Next.js (App Router)
   │  ├─ package.json
   │  ├─ next.config.js
   │  ├─ app/
   │  │  ├─ layout.tsx
   │  │  ├─ page.tsx
   │  │  ├─ ai/page.tsx
   │  │  └─ editor/page.tsx
   │  └─ tsconfig.json
   │
   ├─ mobile/             # Expo (React Native)
   │  ├─ package.json
   │  ├─ app.json
   │  ├─ App.tsx
   │  └─ screens/
   │     ├─ Home.tsx
   │     ├─ AIChat.tsx
   │     ├─ Redeem.tsx
   │     └─ Sketch.tsx
   │
   └─ desktop/            # Electron
      ├─ package.json
      ├─ main.js
      └─ preload.js
  {
  "name": "prepsmart",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ],
  "scripts": {
    "dev:backend": "npm --workspace apps/backend run dev",
    "dev:web": "npm --workspace apps/web run dev",
    "dev:mobile": "npm --workspace apps/mobile run start",
    "dev:desktop": "npm --workspace apps/desktop run start",
    "install-all": "npm install --workspaces"
  }
}
node_modules
.npmrc
dist
.next
.expo
.DS_Store
.env
# copy this to .env at repo root OR to each app that needs it
ORION_MODEL=mock
BACKEND_PORT=5000
NEXT_PUBLIC_BACKEND_URL=http://localhost:5000
MOBILE_BACKEND_URL=http://10.0.2.2:5000
# On iOS simulator use http://localhost:5000
# PrepSmart

Monorepo with:
- Backend (Express)
- Web (Next.js)
- Mobile (Expo)
- Desktop (Electron)
- Shared API helper

## Quick Start

```bash
# 1) install everything
npm run install-all

# 2) start backend
npm run dev:backend
# backend at http://localhost:5000

# 3) start web
npm run dev:web
# web at http://localhost:3000

# 4) start mobile (Expo)
npm run dev:mobile
# press "a" for Android emulator, "i" for iOS

# 5) start desktop (Electron)
npm run dev:desktop
---

## packages/shared

**packages/shared/package.json**
```json
{
  "name": "@prepsmart/shared",
  "version": "1.0.0",
  "main": "src/api.ts",
  "type": "module"
}
export const createApi = (baseUrl: string) => {
  const askAI = async (question: string) => {
    const res = await fetch(`${baseUrl}/api/ai/ask`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ question })
    });
    return res.json() as Promise<{ answer: string }>;
  };

  const redeem = async (code: string) => {
    const res = await fetch(`${baseUrl}/api/redeem`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ code })
    });
    return res.json() as Promise<{ success: boolean; message: string }>;
  };

  return { askAI, redeem };
};
{
  "name": "prepsmart-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "node --watch ./dist/index.js",
    "build": "tsc -p tsconfig.json",
    "predev": "npm run build"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.19.2"
  },
  "devDependencies": {
    "typescript": "^5.4.5"
  }
}
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "outDir": "dist",
    "rootDir": "src",
    "moduleResolution": "Bundler",
    "esModuleInterop": true,
    "strict": true
  },
  "include": ["src"]
}
import express from "express";
import cors from "cors";

import aiRouter from "./routes/ai.js";
import redeemRouter from "./routes/redeem.js";

const app = express();
app.use(cors());
app.use(express.json());

app.use("/api/ai", aiRouter);
app.use("/api/redeem", redeemRouter);

const PORT = Number(process.env.BACKEND_PORT) || 5000;
app.listen(PORT, () => console.log(`Backend running on http://localhost:${PORT}`));
import { Router } from "express";

const router = Router();

/**
 * Orion AI (mock). Swap this with a real LLM later.
 */
router.post("/ask", async (req, res) => {
  const q: string = (req.body?.question || "").toString();
  const answer = q
    ? `Orion: I received your question — "${q}". Here's a helpful starting point.`
    : "Orion: Ask me anything about interviews, resumes, or study planning!";
  res.json({ answer });
});

export default router;
import { Router } from "express";
const router = Router();

const validCodes = new Set(["PREP10", "FREE50", "ORION100"]);

router.post("/", (req, res) => {
  const code: string = (req.body?.code || "").toUpperCase().trim();
  if (validCodes.has(code)) {
    return res.json({ success: true, message: "Code applied successfully!" });
  }
  return res.json({ success: false, message: "Invalid code!" });
});

export default router;
{
  "name": "prepsmart-web",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev -p 3000"
  },
  "dependencies": {
    "@prepsmart/shared": "1.0.0",
    "fabric": "^6.0.0",
    "next": "14.2.3",
    "react": "18.2.0",
    "react-dom": "18.2.0"
  }
}
/** @type {import('next').NextConfig} */
const nextConfig = {};
module.exports = nextConfig;
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "target": "ES2020",
    "module": "ES2020",
    "moduleResolution": "Bundler",
    "strict": true,
    "baseUrl": ".",
    "paths": {}
  }
export const metadata = { title: "PrepSmart" };

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body style={{ fontFamily: "system-ui", margin: 0 }}>
        <div style={{ padding: 16, borderBottom: "1px solid #eee" }}>
          <b>PrepSmart</b> — Web
          <nav style={{ marginTop: 8, display: "flex", gap: 12 }}>
            <a href="/">Home</a>
            <a href="/ai">Orion Chat</a>
            <a href="/editor">Editor</a>
          </nav>
        </div>
        <div style={{ padding: 16 }}>{children}</div>
      </body>
    </html>
  );
}
}
export default function Page() {
  return (
    <main>
      <h2>Welcome to PrepSmart</h2>
      <p>Interviews, resumes, grooming tips, and more — powered by Orion.</p>
    </main>
  );
}
"use client";

import { useState } from "react";
import { createApi } from "@prepsmart/shared/src/api";

const api = createApi(process.env.NEXT_PUBLIC_BACKEND_URL || "http://localhost:5000");

export default function AIPage() {
  const [q, setQ] = useState("");
  const [a, setA] = useState("");

  const ask = async () => {
    const res = await api.askAI(q);
    setA(res.answer);
  };

  return (
    <main>
      <h2>Orion Chat</h2>
      <input
        value={q}
        onChange={(e) => setQ(e.target.value)}
        placeholder="Ask Orion…"
        style={{ padding: 8, width: "60%", maxWidth: 600 }}
      />
      <button onClick={ask} style={{ marginLeft: 8, padding: "8px 12px" }}>Ask</button>
      <pre style={{ marginTop: 16, whiteSpace: "pre-wrap" }}>{a}</pre>
    </main>
  );
}
"use client";

import { useEffect, useRef } from "react";
import { fabric } from "fabric";

export default function EditorPage() {
  const canvasRef = useRef<HTMLCanvasElement | null>(null);
  const fabricRef = useRef<fabric.Canvas | null>(null);

  useEffect(() => {
    if (!canvasRef.current) return;
    const f = new fabric.Canvas(canvasRef.current, { backgroundColor: "#fff" });
    fabricRef.current = f;

    const text = new fabric.IText("PrepSmart", { left: 50, top: 50, fontSize: 32 });
    f.add(text);

    return () => f.dispose();
  }, []);

  const addRect = () => {
    fabricRef.current?.add(new fabric.Rect({ left: 100, top: 120, width: 140, height: 80, fill: "#ddd" }));
  };

  const clearAll = () => fabricRef.current?.clear();

  return (
    <main>
      <h2>Canva-like Editor</h2>
      <div style={{ marginBottom: 8 }}>
        <button onClick={addRect}>Add Rectangle</button>{" "}
        <button onClick={clearAll}>Clear</button>
      </div>
      <canvas ref={canvasRef} width={900} height={500} style={{ border: "1px solid #ccc" }} />
    </main>
  );
}
{
  "name": "prepsmart-mobile",
  "version": "1.0.0",
  "private": true,
  "main": "App.tsx",
  "scripts": {
    "start": "expo start -c"
  },
  "dependencies": {
    "@prepsmart/shared": "1.0.0",
    "expo": "^51.0.0",
    "expo-status-bar": "~1.12.1",
    "react": "18.2.0",
    "react-native": "0.74.0"
  }
}
{
  "expo": {
    "name": "PrepSmart",
    "slug": "prepsmart",
    "scheme": "prepsmart",
    "version": "1.0.0",
    "sdkVersion": "51.0.0",
    "platforms": ["ios", "android"],
    "orientation": "portrait"
  }
}
import { StatusBar } from "expo-status-bar";
import React from "react";
import { SafeAreaView, View, Button } from "react-native";
import Home from "./screens/Home";
import AIChat from "./screens/AIChat";
import Redeem from "./screens/Redeem";
import Sketch from "./screens/Sketch";

export default function App() {
  const [screen, setScreen] = React.useState<"home"|"ai"|"redeem"|"sketch">("home");
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <View style={{ padding: 12, flexDirection: "row", gap: 8 }}>
        <Button title="Home" onPress={() => setScreen("home")} />
        <Button title="Orion" onPress={() => setScreen("ai")} />
        <Button title="Redeem" onPress={() => setScreen("redeem")} />
        <Button title="Sketch" onPress={() => setScreen("sketch")} />
      </View>
      {screen === "home" && <Home/>}
      {screen === "ai" && <AIChat/>}
      {screen === "redeem" && <Redeem/>}
      {screen === "sketch" && <Sketch/>}
      <StatusBar style="auto" />
    </SafeAreaView>
  );
}
import React from "react";
import { View, Text } from "react-native";

export default function Home() {
  return (
    <View style={{ flex: 1, alignItems: "center", justifyContent: "center" }}>
      <Text style={{ fontSize: 20, fontWeight: "600" }}>Welcome to PrepSmart Mobile</Text>
    </View>
  );
}
import React, { useState } from "react";
import { View, TextInput, Button, Text } from "react-native";
import { createApi } from "@prepsmart/shared/src/api";

const api = createApi(process.env.MOBILE_BACKEND_URL || "http://10.0.2.2:5000");

export default function AIChat() {
  const [q, setQ] = useState("");
  const [a, setA] = useState("");

  const ask = async () => {
    const res = await api.askAI(q);
    setA(res.answer);
  };

  return (
    <View style={{ padding: 16 }}>
      <TextInput
        placeholder="Ask Orion…"
        value={q}
        onChangeText={setQ}
        style={{ borderWidth: 1, padding: 8, borderRadius: 6 }}
      />
      <Button title="Ask" onPress={ask} />
      <Text style={{ marginTop: 12 }}>{a}</Text>
    </View>
  );
}
import React, { useState } from "react";
import { View, TextInput, Button, Text } from "react-native";
import { createApi } from "@prepsmart/shared/src/api";

const api = createApi(process.env.MOBILE_BACKEND_URL || "http://10.0.2.2:5000");

export default function Redeem() {
  const [code, setCode] = useState("");
  const [msg, setMsg] = useState("");

  const redeem = async () => {
    const res = await api.redeem(code);
    setMsg(res.message);
  };

  return (
    <View style={{ padding: 16 }}>
      <TextInput
        placeholder="Enter code"
        value={code}
        onChangeText={setCode}
        style={{ borderWidth: 1, padding: 8, borderRadius: 6 }}
      />
      <Button title="Redeem" onPress={redeem} />
      <Text style={{ marginTop: 12 }}>{msg}</Text>
    </View>
  );
}
import React, { useRef } from "react";
import { View, Button } from "react-native";
import { Canvas, Path, Skia, useCanvasRef } from "@shopify/react-native-skia";

/**
 * Lightweight sketch using Skia (no native linking hassles with Expo).
 * If @shopify/react-native-skia is heavy for you, you can replace with a simple View or 3rd-party sketch lib.
 */
export default function Sketch() {
  const ref = useCanvasRef();
  const pathRef = useRef(Path.Make());

  const clear = () => {
    pathRef.current = Path.Make();
    ref.current?.redraw();
  };

  return (
    <View style={{ flex: 1 }}>
      <Canvas ref={ref} style={{ flex: 1 }}>
        {/* Empty canvas — draw features can be expanded later */}
      </Canvas>
      <Button title="Clear" onPress={clear} />
    </View>
  );
}
{
  "name": "prepsmart-desktop",
  "version": "1.0.0",
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  },
  "dependencies": {
    "electron": "^29.4.0"
  }
}
const { app, BrowserWindow } = require("electron");
const path = require("path");

function createWindow() {
  const win = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: { preload: path.join(__dirname, "preload.js") }
  });
  // Load the web app (assumes it's running locally)
  win.loadURL("http://localhost:3000");
}
// reserved for secure APIs if needed later

app.whenReady().then(createWindow);
cd PrepSmart
cp .env.example .env  # (optional; adjust URLs if needed)
npm run install-all
npm run dev:backend   # tab 1
npm run dev:web       # tab 2
npm run dev:mobile    # tab 3 (press a/i to open emulator)
npm run dev:desktop   # tab 4
git add .
git commit -m "Add full PrepSmart monorepo (backend, web, mobile, desktop)"
git push origin main
