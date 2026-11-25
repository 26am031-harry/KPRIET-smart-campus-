# KPRIET-smart-campus-
i created an web app as"KPRIET smart campus" to find the lost things and rapert the maintance issue on- campus

#!/usr/bin/env node
import fs from "node:fs";
import path from "node:path";
import { execSync } from "node:child_process";

const repoUrl = process.argv[2];
const branch = process.argv[3] || "main";

if (!repoUrl) {
  console.error("Usage: node scripts/publish-to-github.mjs <github-repo-url> [branch]");
  console.error("Example: node scripts/publish-to-github.mjs https://github.com/your-username/smart-campus-hub.git main");
  process.exit(1);
}

const projectFiles = {
  ".gitignore": `node_modules/
.next/
out/
dist/
.env
.env*.local
*.tsbuildinfo
next-env.d.ts
`,

  ".env.example": `# Smart Campus Management System Environment Example
NEXT_PUBLIC_APP_NAME="Smart Campus Management System"
DATABASE_URL="postgresql://user:password@localhost:5432/smart_campus"
`,

  "README.md": `# Smart Campus Management System

Unified Next.js (App Router) + TypeScript + Tailwind CSS application combining:
- Campus Feed, Browse, Post Details, Claims, Comments, Photo Picker & Zoomable Image
- Escalation Queue & Admin Status Controls
- Authentication & Personal Hub
- Campus Zone Location Tracker & Assistant
`,

  "src/lib/campus-store.ts": `export type PostCategory = "Lost & Found" | "Facility Issue" | "Event & Notice" | "Safety";
export type PostStatus = "Open" | "Claimed" | "Escalated" | "Resolved";
export type UserRole = "student" | "faculty" | "admin";

export interface CampusUser {
  id: string;
  name: string;
  email: string;
  rollOrStaffId: string;
  role: UserRole;
  department: string;
}

export interface ClaimItem {
  id: string;
  postId: string;
  claimantName: string;
  claimantEmail: string;
  proofNote: string;
  status: "Pending" | "Approved" | "Rejected";
  createdAt: string;
}

export interface CommentItem {
  id: string;
  postId: string;
  authorName: string;
  message: string;
  createdAt: string;
}

export interface EscalationItem {
  id: string;
  postId: string;
  title: string;
  zone: string;
  severity: "Low" | "Medium" | "High" | "Critical";
  assignedTo: string;
  status: "Open" | "In Progress" | "Resolved";
  notes: string;
  updatedAt: string;
}

export interface CampusPost {
  id: string;
  title: string;
  description: string;
  category: PostCategory;
  status: PostStatus;
  zone: string;
  authorName: string;
  authorEmail: string;
  upvotes: number;
  photoUrl?: string;
  createdAt: string;
}

export interface CampusState {
  currentUser: CampusUser | null;
  users: CampusUser[];
  posts: CampusPost[];
  claims: ClaimItem[];
  comments: CommentItem[];
  escalations: EscalationItem[];
  upvotedPostIds: string[];
  activeZone: string;
}

export const CAMPUS_ZONES = [
  "Main Library Block",
  "CSE & IT Wing",
  "Central Canteen",
  "Boys Hostel A",
  "Girls Hostel B",
  "Sports Complex",
  "Auditorium & Seminar Hall",
];

export const SAMPLE_PHOTOS = [
  {
    label: "Library Study Desk",
    url: "https://images.unsplash.com/photo-1521587760476-6c12a4b040da?auto=format&fit=crop&w=900&q=80",
  },
  {
    label: "Campus Corridor & Notice Board",
    url: "https://images.unsplash.com/photo-1562774053-701939374585?auto=format&fit=crop&w=900&q=80",
  },
  {
    label: "Tech Lab & Workstations",
    url: "https://images.unsplash.com/photo-1517245386807-bb43f82c33c4?auto=format&fit=crop&w=900&q=80",
  },
];

export const initialCampusState: CampusState = {
  currentUser: {
    id: "u-1",
    name: "Ananya Krishnan",
    email: "ananya.k@campus.edu.in",
    rollOrStaffId: "23CS042",
    role: "student",
    department: "Computer Science & Engineering",
  },
  users: [
    {
      id: "u-1",
      name: "Ananya Krishnan",
      email: "ananya.k@campus.edu.in",
      rollOrStaffId: "23CS042",
      role: "student",
      department: "Computer Science & Engineering",
    },
    {
      id: "u-2",
      name: "Dr. Rajeshwar Nair",
      email: "admin@campus.edu.in",
      rollOrStaffId: "FAC-108",
      role: "admin",
      department: "Campus Operations & Warden Office",
    },
  ],
  posts: [
    {
      id: "p-101",
      title: "Black Laptop Sleeve & Charger Left Near Reading Room",
      description:
        "Found a matte black 15-inch sleeve with a 65W USB-C adapter on table 14 of the second-floor reading room around 4:30 PM.",
      category: "Lost & Found",
      status: "Open",
      zone: "Main Library Block",
      authorName: "Karthik Subramanian",
      authorEmail: "karthik.s@campus.edu.in",
      upvotes: 14,
      photoUrl: SAMPLE_PHOTOS[0].url,
      createdAt: "Today, 4:45 PM",
    },
    {
      id: "p-102",
      title: "Projector & AC Tripping in Seminar Hall B",
      description:
        "The main presentation projector shuts down after 15 minutes and cooling is weak ahead of tomorrow's workshop.",
      category: "Facility Issue",
      status: "Escalated",
      zone: "Auditorium & Seminar Hall",
      authorName: "Meera Chandran",
      authorEmail: "meera.c@campus.edu.in",
      upvotes: 23,
      photoUrl: SAMPLE_PHOTOS[2].url,
      createdAt: "Today, 2:15 PM",
    },
    {
      id: "p-103",
      title: "Inter-Department Hackathon & Open Demo Registrations",
      description:
        "Teams of 2 to 4 can register for the 24-hour campus innovation sprint. Food and Wi-Fi passes are covered.",
      category: "Event & Notice",
      status: "Open",
      zone: "CSE & IT Wing",
      authorName: "Dr. Rajeshwar Nair",
      authorEmail: "admin@campus.edu.in",
      upvotes: 31,
      photoUrl: SAMPLE_PHOTOS[1].url,
      createdAt: "Yesterday",
    },
    {
      id: "p-104",
      title: "Dim Streetlight Along Hostel Walkway Near Gate 2",
      description:
        "Two pole lamps along the evening pedestrian path have been flickering after rain.",
      category: "Safety",
      status: "Open",
      zone: "Girls Hostel B",
      authorName: "Divya Menon",
      authorEmail: "divya.m@campus.edu.in",
      upvotes: 19,
      createdAt: "Yesterday",
    },
  ],
  claims: [
    {
      id: "c-201",
      postId: "p-101",
      claimantName: "Siddharth Verma",
      claimantEmail: "siddharth.v@campus.edu.in",
      proofNote: "Sleeve has a small blue sticker inside the front pocket and a braided cable.",
      status: "Pending",
      createdAt: "Today, 5:10 PM",
    },
  ],
  comments: [
    {
      id: "cm-301",
      postId: "p-101",
      authorName: "Library Desk",
      message: "If unclaimed by 8:00 PM, the item will be logged at the Main Security Desk.",
      createdAt: "Today, 5:00 PM",
    },
    {
      id: "cm-302",
      postId: "p-102",
      authorName: "Electrical Maintenance",
      message: "MCB inspection scheduled for early morning before the session starts.",
      createdAt: "Today, 3:05 PM",
    },
  ],
  escalations: [
    {
      id: "e-401",
      postId: "p-102",
      title: "Seminar Hall B Projector & AC Load Fault",
      zone: "Auditorium & Seminar Hall",
      severity: "High",
      assignedTo: "Campus Electrical & AV Team",
      status: "In Progress",
      notes: "Checking phase voltage and projector thermal cut-off.",
      updatedAt: "Today, 3:20 PM",
    },
    {
      id: "e-402",
      postId: "p-104",
      title: "Hostel Walkway Lighting Near Gate 2",
      zone: "Girls Hostel B",
      severity: "Critical",
      assignedTo: "Estate & Security Operations",
      status: "Open",
      notes: "Night patrol flagged low visibility on two poles.",
      updatedAt: "Yesterday",
    },
  ],
  upvotedPostIds: ["p-102"],
  activeZone: "Main Library Block",
};
`,

  "src/app/api/health/route.ts": `import { NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({
    status: "ok",
    service: "smart-campus-management-system",
    timestamp: new Date().toISOString(),
  });
}
`,

  "src/app/api/stats/route.ts": `import { NextResponse } from "next/server";
import { initialCampusState } from "@/lib/campus-store";

export async function GET() {
  const { posts, claims, escalations } = initialCampusState;
  return NextResponse.json({
    totalPosts: posts.length,
    openPosts: posts.filter((p) => p.status === "Open").length,
    pendingClaims: claims.filter((c) => c.status === "Pending").length,
    activeEscalations: escalations.filter((e) => e.status !== "Resolved").length,
  });
}
`,

  "src/app/api/assistant/route.ts": `import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  const body = await req.json().catch(() => ({}));
  const query = String(body?.query ?? "").trim();
  const zone = String(body?.zone ?? "Campus");

  if (!query) {
    return NextResponse.json({ reply: "Ask me about lost items, facility escalations, claims, or campus zones." });
  }

  const lower = query.toLowerCase();
  let reply = \`For \${zone}: you can browse active posts, submit a new report, or track open escalations from the top navigation bar.\`;

  if (lower.includes("lost") || lower.includes("found") || lower.includes("claim")) {
    reply = \`To recover a lost item in \${zone}, open the matching post, submit a claim with specific identifying details, and coordinate pickup at the security or library desk.\`;
  } else if (lower.includes("escalat") || lower.includes("broken") || lower.includes("repair") || lower.includes("ac") || lower.includes("light")) {
    reply = \`Urgent facility or safety issues in \${zone} can be escalated directly from a post or managed in the Admin Escalation Queue with severity and assignee tracking.\`;
  } else if (lower.includes("emergency") || lower.includes("safety") || lower.includes("help")) {
    reply = \`For immediate campus safety or medical emergencies in India, dial 112 (or 108 for ambulance) and notify Campus Security at \${zone}.\`;
  }

  return NextResponse.json({ reply });
}
`,

  "src/app/api/auth/login/route.ts": `import { NextRequest, NextResponse } from "next/server";
import { initialCampusState } from "@/lib/campus-store";

export async function POST(req: NextRequest) {
  const { email, role } = await req.json().catch(() => ({}));
  const matched =
    initialCampusState.users.find((u) => u.email.toLowerCase() === String(email || "").toLowerCase()) ||
    initialCampusState.users[0];

  return NextResponse.json({
    user: {
      ...matched,
      email: email || matched.email,
      role: role || matched.role,
    },
  });
}
`,

  "src/app/api/auth/register/route.ts": `import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  const { name, email, rollOrStaffId, department, role } = await req.json().catch(() => ({}));
  if (!name || !email) {
    return NextResponse.json({ error: "Name and email are required." }, { status: 400 });
  }

  return NextResponse.json(
    {
      user: {
        id: \`u-\${Date.now()}\`,
        name,
        email,
        rollOrStaffId: rollOrStaffId || "CAMPUS-NEW",
        department: department || "General Campus",
        role: role || "student",
      },
    },
    { status: 201 }
  );
}
`,

  "src/app/api/auth/logout/route.ts": `import { NextResponse } from "next/server";

export async function POST() {
  return NextResponse.json({ loggedOut: true });
}
`,

  "src/app/api/auth/me/route.ts": `import { NextResponse } from "next/server";
import { initialCampusState } from "@/lib/campus-store";

export async function GET() {
  return NextResponse.json({ user: initialCampusState.currentUser });
}
`,

  "src/components/CampusModules.tsx": `"use client";

import React, { useState } from "react";
import {
  Bell,
  Camera,
  CheckCircle2,
  ChevronRight,
  Compass,
  FilePlus2,
  LayoutDashboard,
  LogIn,
  LogOut,
  MapPin,
  MessageSquare,
  Mic,
  MicOff,
  Search,
  Send,
  ShieldAlert,
  ThumbsUp,
  UserCheck,
  X,
  ZoomIn,
} from "lucide-react";
import {
  CAMPUS_ZONES,
  CampusPost,
  CampusUser,
  ClaimItem,
  CommentItem,
  EscalationItem,
  PostCategory,
  PostStatus,
  SAMPLE_PHOTOS,
} from "@/lib/campus-store";

export type ActiveTab = "home" | "browse" | "submit" | "post" | "admin" | "my" | "login" | "register";

export function Navbar({
  activeTab,
  onNavigate,
  currentUser,
  onLogout,
  escalationCount,
}: {
  activeTab: ActiveTab;
  onNavigate: (tab: ActiveTab) => void;
  currentUser: CampusUser | null;
  onLogout: () => void;
  escalationCount: number;
}) {
  const navItems: { id: ActiveTab; label: string; icon: React.ReactNode; badge?: number }[] = [
    { id: "home", label: "Overview", icon: <LayoutDashboard className="w-4 h-4" /> },
    { id: "browse", label: "Browse Feed", icon: <Search className="w-4 h-4" /> },
    { id: "submit", label: "New Post", icon: <FilePlus2 className="w-4 h-4" /> },
    { id: "my", label: "My Hub", icon: <UserCheck className="w-4 h-4" /> },
    {
      id: "admin",
      label: "Escalations & Admin",
      icon: <ShieldAlert className="w-4 h-4" />,
      badge: escalationCount,
    },
  ];

  return (
    <header className="sticky top-0 z-30 bg-white/95 backdrop-blur border-b border-slate-200">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 h-16 flex items-center justify-between gap-3">
        <button
          onClick={() => onNavigate("home")}
          className="flex items-center gap-2.5 text-left font-bold text-slate-900"
        >
          <span className="w-9 h-9 rounded-xl bg-indigo-600 text-white flex items-center justify-center font-semibold">
            SC
          </span>
          <div>
            <span className="text-base tracking-tight block leading-none">Smart Campus</span>
            <span className="text-[11px] text-slate-500 font-normal">Unified Operations & Feed</span>
          </div>
        </button>

        <nav className="hidden md:flex items-center gap-1">
          {navItems.map((item) => {
            const active = activeTab === item.id;
            return (
              <button
                key={item.id}
                onClick={() => onNavigate(item.id)}
                className={\`px-3 py-2 rounded-lg text-sm font-medium flex items-center gap-1.5 transition \${
                  active
                    ? "bg-indigo-50 text-indigo-700"
                    : "text-slate-600 hover:bg-slate-100 hover:text-slate-900"
                }\`}
              >
                {item.icon}
                <span>{item.label}</span>
                {typeof item.badge === "number" && item.badge > 0 && (
                  <span className="ml-0.5 px-1.5 py-0.5 text-xs rounded-full bg-amber-100 text-amber-800 font-semibold">
                    {item.badge}
                  </span>
                )}
              </button>
            );
          })}
        </nav>

        <div className="flex items-center gap-2">
          {currentUser ? (
            <>
              <div className="hidden sm:block text-right mr-1">
                <div className="text-xs font-semibold text-slate-800">{currentUser.name}</div>
                <div className="text-[11px] text-slate-500 uppercase">{currentUser.role}</div>
              </div>
              <button
                onClick={onLogout}
                className="px-3 py-1.5 rounded-lg border border-slate-200 text-xs font-medium text-slate-700 hover:bg-slate-50 flex items-center gap-1.5"
              >
                <LogOut className="w-3.5 h-3.5" />
                Sign out
              </button>
            </>
          ) : (
            <button
              onClick={() => onNavigate("login")}
              className="px-3 py-1.5 rounded-lg bg-indigo-600 text-white text-xs font-medium hover:bg-indigo-700 flex items-center gap-1.5"
            >
              <LogIn className="w-3.5 h-3.5" />
              Sign in
            </button>
          )}
        </div>
      </div>

      <div className="flex md:hidden overflow-x-auto border-t border-slate-100 px-3 py-1.5 gap-1 bg-slate-50/70">
        {navItems.map((item) => (
          <button
            key={item.id}
            onClick={() => onNavigate(item.id)}
            className={\`px-2.5 py-1.5 rounded-md text-xs font-medium whitespace-nowrap flex items-center gap-1 \${
              activeTab === item.id ? "bg-indigo-600 text-white" : "text-slate-600 bg-white border border-slate-200"
            }\`}
          >
            {item.icon}
            {item.label}
          </button>
        ))}
      </div>
    </header>
  );
}

export function FilterBar({
  searchQuery,
  onSearchChange,
  category,
  onCategoryChange,
  status,
  onStatusChange,
  zone,
  onZoneChange,
}: {
  searchQuery: string;
  onSearchChange: (v: string) => void;
  category: string;
  onCategoryChange: (v: string) => void;
  status: string;
  onStatusChange: (v: string) => void;
  zone: string;
  onZoneChange: (v: string) => void;
}) {
  return (
    <div className="bg-white rounded-xl border border-slate-200 p-4 grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3">
      <div className="relative">
        <Search className="w-4 h-4 text-slate-400 absolute left-3 top-1/2 -translate-y-1/2" />
        <input
          type="text"
          value={searchQuery}
          onChange={(e) => onSearchChange(e.target.value)}
          placeholder="Search posts, items, issues..."
          className="w-full pl-9 pr-3 py-2 text-sm rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500"
        />
      </div>

      <select
        value={category}
        onChange={(e) => onCategoryChange(e.target.value)}
        className="px-3 py-2 text-sm rounded-lg border border-slate-200 bg-white focus:outline-none focus:ring-2 focus:ring-indigo-500"
      >
        <option value="All">All Categories</option>
        <option value="Lost & Found">Lost & Found</option>
        <option value="Facility Issue">Facility Issue</option>
        <option value="Event & Notice">Event & Notice</option>
        <option value="Safety">Safety</option>
      </select>

      <select
        value={status}
        onChange={(e) => onStatusChange(e.target.value)}
        className="px-3 py-2 text-sm rounded-lg border border-slate-200 bg-white focus:outline-none focus:ring-2 focus:ring-indigo-500"
      >
        <option value="All">All Statuses</option>
        <option value="Open">Open</option>
        <option value="Claimed">Claimed</option>
        <option value="Escalated">Escalated</option>
        <option value="Resolved">Resolved</option>
      </select>

      <select
        value={zone}
        onChange={(e) => onZoneChange(e.target.value)}
        className="px-3 py-2 text-sm rounded-lg border border-slate-200 bg-white focus:outline-none focus:ring-2 focus:ring-indigo-500"
      >
        <option value="All">All Campus Zones</option>
        {CAMPUS_ZONES.map((z) => (
          <option key={z} value={z}>
            {z}
          </option>
        ))}
      </select>
    </div>
  );
}

const statusBadgeStyle: Record<PostStatus, string> = {
  Open: "bg-emerald-50 text-emerald-700 border-emerald-200",
  Claimed: "bg-sky-50 text-sky-700 border-sky-200",
  Escalated: "bg-amber-50 text-amber-800 border-amber-200",
  Resolved: "bg-slate-100 text-slate-600 border-slate-200",
};

export function PostCard({
  post,
  hasUpvoted,
  onUpvote,
  onOpenPost,
  commentCount,
  claimCount,
}: {
  post: CampusPost;
  hasUpvoted: boolean;
  onUpvote: (id: string) => void;
  onOpenPost: (id: string) => void;
  commentCount: number;
  claimCount: number;
}) {
  return (
    <article className="bg-white rounded-xl border border-slate-200 p-4 hover:border-indigo-300 transition flex flex-col justify-between gap-4">
      <div className="space-y-2.5">
        <div className="flex items-center justify-between gap-2 flex-wrap">
          <span className="text-xs font-semibold px-2.5 py-0.5 rounded-full bg-indigo-50 text-indigo-700">
            {post.category}
          </span>
          <span className={\`text-xs font-medium px-2.5 py-0.5 rounded-full border \${statusBadgeStyle[post.status]}\`}>
            {post.status}
          </span>
        </div>

        <button onClick={() => onOpenPost(post.id)} className="text-left group block w-full">
          <h3 className="font-semibold text-slate-900 group-hover:text-indigo-600 transition line-clamp-2">
            {post.title}
          </h3>
        </button>

        <p className="text-sm text-slate-600 line-clamp-2">{post.description}</p>

        {post.photoUrl && (
          <div
            onClick={() => onOpenPost(post.id)}
            className="h-36 w-full rounded-lg overflow-hidden bg-slate-100 cursor-pointer"
          >
            <img src={post.photoUrl} alt={post.title} className="w-full h-full object-cover" />
          </div>
        )}
      </div>

      <div className="pt-3 border-t border-slate-100 flex items-center justify-between text-xs text-slate-500">
        <div className="flex items-center gap-3">
          <button
            onClick={() => onUpvote(post.id)}
            className={\`inline-flex items-center gap-1 px-2 py-1 rounded-md border font-medium transition \${
              hasUpvoted
                ? "bg-indigo-50 border-indigo-200 text-indigo-700"
                : "bg-white border-slate-200 text-slate-600 hover:bg-slate-50"
            }\`}
          >
            <ThumbsUp className="w-3.5 h-3.5" />
            <span>{post.upvotes}</span>
          </button>
          <span className="inline-flex items-center gap-1">
            <MessageSquare className="w-3.5 h-3.5" />
            {commentCount}
          </span>
          {post.category === "Lost & Found" && (
            <span className="inline-flex items-center gap-1 text-sky-700 font-medium">
              {claimCount} claim{claimCount === 1 ? "" : "s"}
            </span>
          )}
        </div>

        <button
          onClick={() => onOpenPost(post.id)}
          className="inline-flex items-center gap-0.5 font-semibold text-indigo-600 hover:text-indigo-800"
        >
          Details <ChevronRight className="w-3.5 h-3.5" />
        </button>
      </div>
    </article>
  );
}

export function PhotoPicker({
  value,
  onChange,
}: {
  value?: string;
  onChange: (url: string) => void;
}) {
  return (
    <div className="space-y-2">
      <label className="text-xs font-semibold text-slate-700 flex items-center gap-1.5">
        <Camera className="w-3.5 h-3.5 text-indigo-600" />
        Attach Campus Photo (Optional)
      </label>
      <div className="grid grid-cols-3 gap-2">
        {SAMPLE_PHOTOS.map((item) => {
          const selected = value === item.url;
          return (
            <button
              key={item.url}
              type="button"
              onClick={() => onChange(selected ? "" : item.url)}
              className={\`rounded-lg overflow-hidden border text-left transition \${
                selected ? "border-indigo-600 ring-2 ring-indigo-500/30" : "border-slate-200 hover:border-slate-300"
              }\`}
            >
              <img src={item.url} alt={item.label} className="w-full h-16 object-cover" />
              <div className="p-1.5 text-[11px] font-medium text-slate-700 truncate">{item.label}</div>
            </button>
          );
        })}
      </div>
    </div>
  );
}

export function ZoomableImage({ src, alt }: { src: string; alt: string }) {
  const [zoomed, setZoomed] = useState(false);

  return (
    <div className="space-y-2">
      <div className="relative rounded-xl overflow-hidden border border-slate-200 bg-slate-100">
        <img
          src={src}
          alt={alt}
          className={\`w-full transition-transform duration-200 \${
            zoomed ? "h-96 object-contain bg-slate-900 scale-105" : "h-64 object-cover"
          }\`}
        />
        <button
          type="button"
          onClick={() => setZoomed((z) => !z)}
          className="absolute bottom-3 right-3 px-2.5 py-1.5 rounded-lg bg-slate-900/80 text-white text-xs font-medium flex items-center gap-1 hover:bg-slate-900"
        >
          <ZoomIn className="w-3.5 h-3.5" />
          {zoomed ? "Reset View" : "Inspect"}
        </button>
      </div>
    </div>
  );
}

export function SubmitForm({
  currentUser,
  defaultZone,
  onCreatePost,
}: {
  currentUser: CampusUser | null;
  defaultZone: string;
  onCreatePost: (post: Omit<CampusPost, "id" | "upvotes" | "createdAt" | "status">) => void;
}) {
  const [title, setTitle] = useState("");
  const [category, setCategory] = useState<PostCategory>("Lost & Found");
  const [zone, setZone] = useState(defaultZone || CAMPUS_ZONES[0]);
  const [description, setDescription] = useState("");
  const [photoUrl, setPhotoUrl] = useState("");
  const [error, setError] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!title.trim() || !description.trim()) {
      setError("Please enter both a clear title and description.");
      return;
    }
    setError("");
    onCreatePost({
      title: title.trim(),
      category,
      zone,
      description: description.trim(),
      photoUrl: photoUrl || undefined,
      authorName: currentUser?.name || "Campus Guest",
      authorEmail: currentUser?.email || "guest@campus.edu.in",
    });
    setTitle("");
    setDescription("");
    setPhotoUrl("");
  };

  return (
    <form onSubmit={handleSubmit} className="bg-white rounded-xl border border-slate-200 p-6 space-y-4">
      <div>
        <h2 className="text-lg font-bold text-slate-900">Submit a Campus Post</h2>
        <p className="text-sm text-slate-500">Report a lost/found item, facility fault, safety alert, or campus notice.</p>
      </div>

      {error && <div className="p-3 rounded-lg bg-rose-50 text-rose-700 text-xs font-medium">{error}</div>}

      <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <div>
          <label className="block text-xs font-semibold text-slate-700 mb-1">Category</label>
          <select
            value={category}
            onChange={(e) => setCategory(e.target.value as PostCategory)}
            className="w-full px-3 py-2 text-sm rounded-lg border border-slate-200 bg-white"
          >
            <option value="Lost & Found">Lost & Found</option>
            <option value="Facility Issue">Facility Issue</option>
            <option value="Event & Notice">Event & Notice</option>
            <option value="Safety">Safety</option>
          </select>
        </div>

        <div>
          <label className="block text-xs font-semibold text-slate-700 mb-1">Campus Zone</label>
          <select
            value={zone}
            onChange={(e) => setZone(e.target.value)}
            className="w-full px-3 py-2 text-sm rounded-lg border border-slate-200 bg-white"
          >
            {CAMPUS_ZONES.map((z) => (
              <option key={z} value={z}>
                {z}
              </option>
            ))}
          </select>
        </div>
      </div>

      <div>
        <label className="block text-xs font-semibold text-slate-700 mb-1">Title</label>
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="e.g., Blue Scientific Calculator Found in Lab 2"
          className="w-full px-3 py-2 text-sm rounded-lg border border-slate-200"
        />
      </div>

      <div>
        <label className="block text-xs font-semibold text-slate-700 mb-1">Details</label>
        <textarea
          rows={4}
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          placeholder="Add time, exact spot, identifying marks, or urgency details..."
          className="w-full px-3 py-2 text-sm rounded-lg border border-slate-200"
        />
      </div>

      <PhotoPicker value={photoUrl} onChange={setPhotoUrl} />

      <button
        type="submit"
        className="w-full sm:w-auto px-5 py-2.5 rounded-lg bg-indigo-600 text-white text-sm font-semibold hover:bg-indigo-700 transition"
      >
        Publish Post
      </button>
    </form>
  );
}

export function PostActions({
  post,
  currentUser,
  claims,
  comments,
  onAddClaim,
  onAddComment,
  onEscalatePost,
}: {
  post: CampusPost;
  currentUser: CampusUser | null;
  claims: ClaimItem[];
  comments: CommentItem[];
  onAddClaim: (postId: string, proofNote: string) => void;
  onAddComment: (postId: string, message: string) => void;
  onEscalatePost: (post: CampusPost, severity: EscalationItem["severity"], assignedTo: string) => void;
}) {
  const [proofNote, setProofNote] = useState("");
  const [commentText, setCommentText] = useState("");
  const [severity, setSeverity] = useState<EscalationItem["severity"]>("High");
  const [assignee, setAssignee] = useState("Campus Facilities & Security");

  return (
    <div className="space-y-6">
      <div className="bg-white rounded-xl border border-slate-200 p-5 space-y-4">
        <h3 className="font-semibold text-slate-900">Ownership / Verification Claims ({claims.length})</h3>
        <div className="space-y-2.5">
          {claims.length === 0 ? (
            <p className="text-sm text-slate-500">No claims submitted for this post yet.</p>
          ) : (
            claims.map((c) => (
              <div key={c.id} className="p-3 rounded-lg bg-slate-50 border border-slate-200 text-sm space-y-1">
                <div className="flex items-center justify-between">
                  <span className="font-semibold text-slate-800">{c.claimantName}</span>
                  <span className="text-xs px-2 py-0.5 rounded-full bg-amber-100 text-amber-800 font-medium">
                    {c.status}
                  </span>
                </div>
                <p className="text-slate-600">{c.proofNote}</p>
                <div className="text-[11px] text-slate-400">{c.createdAt}</div>
              </div>
            ))
          )}
        </div>

        <form
          onSubmit={(e) => {
            e.preventDefault();
            if (!proofNote.trim()) return;
            onAddClaim(post.id, proofNote.trim());
            setProofNote("");
          }}
          className="flex flex-col sm:flex-row gap-2"
        >
          <input
            type="text"
            value={proofNote}
            onChange={(e) => setProofNote(e.target.value)}
            placeholder="Describe proof of ownership or context..."
            className="flex-1 px-3 py-2 text-sm rounded-lg border border-slate-200"
          />
          <button
            type="submit"
            className="px-4 py-2 rounded-lg bg-sky-600 text-white text-sm font-medium hover:bg-sky-700"
          >
            Submit Claim
          </button>
        </form>
      </div>

      <div className="bg-white rounded-xl border border-slate-200 p-5 space-y-3">
        <h3 className="font-semibold text-slate-900 flex items-center gap-2">
          <ShieldAlert className="w-4 h-4 text-amber-600" />
          Escalate to Campus Operations
        </h3>
        <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
          <select
            value={severity}
            onChange={(e) => setSeverity(e.target.value as EscalationItem["severity"])}
            className="px-3 py-2 text-sm rounded-lg border border-slate-200 bg-white"
          >
            <option value="Low">Low Severity</option>
            <option value="Medium">Medium Severity</option>
            <option value="High">High Severity</option>
            <option value="Critical">Critical Severity</option>
          </select>
          <input
            type="text"
            value={assignee}
            onChange={(e) => setAssignee(e.target.value)}
            placeholder="Assigned team or office"
            className="px-3 py-2 text-sm rounded-lg border border-slate-200"
          />
        </div>
        <button
          type="button"
          onClick={() => onEscalatePost(post, severity, assignee)}
          className="px-4 py-2 rounded-lg bg-amber-600 text-white text-sm font-medium hover:bg-amber-700"
        >
          Escalate Post
        </button>
      </div>

      <div className="bg-white rounded-xl border border-slate-200 p-5 space-y-4">
        <h3 className="font-semibold text-slate-900">Updates & Comments ({comments.length})</h3>
        <div className="space-y-2.5">
          {comments.map((cm) => (
            <div key={cm.id} className="p-3 rounded-lg bg-slate-50 border border-slate-100">
              <div className="flex items-center justify-between text-xs text-slate-500 mb-1">
                <span className="font-semibold text-slate-700">{cm.authorName}</span>
                <span>{cm.createdAt}</span>
              </div>
              <p className="text-sm text-slate-700">{cm.message}</p>
            </div>
          ))}
        </div>

        <form
          onSubmit={(e) => {
            e.preventDefault();
            if (!commentText.trim()) return;
            onAddComment(post.id, commentText.trim());
            setCommentText("");
          }}
          className="flex gap-2"
        >
          <input
            type="text"
            value={commentText}
            onChange={(e) => setCommentText(e.target.value)}
            placeholder={\`Comment as \${currentUser?.name || "Campus User"}...\`}
            className="flex-1 px-3 py-2 text-sm rounded-lg border border-slate-200"
          />
          <button
            type="submit"
            className="px-4 py-2 rounded-lg bg-indigo-600 text-white text-sm font-medium hover:bg-indigo-700"
          >
            Post
          </button>
        </form>
      </div>
    </div>
  );
}

export function EscalationQueue({
  escalations,
  onUpdateEscalationStatus,
}: {
  escalations: EscalationItem[];
  onUpdateEscalationStatus: (id: string, status: EscalationItem["status"]) => void;
}) {
  return (
    <div className="bg-white rounded-xl border border-slate-200 divide-y divide-slate-100">
      {escalations.length === 0 ? (
        <div className="p-6 text-sm text-slate-500">No active escalations right now.</div>
      ) : (
        escalations.map((item) => (
          <div key={item.id} className="p-4 flex flex-col sm:flex-row sm:items-center justify-between gap-4">
            <div className="space-y-1">
              <div className="flex items-center gap-2 flex-wrap">
                <span className="font-semibold text-slate-900">{item.title}</span>
                <span className="text-xs px-2 py-0.5 rounded-full bg-rose-50 text-rose-700 font-medium">
                  {item.severity}
                </span>
                <span className="text-xs px-2 py-0.5 rounded-full bg-slate-100 text-slate-700 font-medium">
                  {item.status}
                </span>
              </div>
              <p className="text-xs text-slate-500">
                Zone: <strong className="text-slate-700">{item.zone}</strong> • Assigned to:{" "}
                <strong className="text-slate-700">{item.assignedTo}</strong>
              </p>
              <p className="text-sm text-slate-600">{item.notes}</p>
            </div>

            <div className="flex items-center gap-2 shrink-0">
              {(["Open", "In Progress", "Resolved"] as const).map((st) => (
                <button
                  key={st}
                  onClick={() => onUpdateEscalationStatus(item.id, st)}
                  className={\`px-2.5 py-1.5 rounded-lg text-xs font-medium border transition \${
                    item.status === st
                      ? "bg-indigo-600 text-white border-indigo-600"
                      : "bg-white text-slate-700 border-slate-200 hover:bg-slate-50"
                  }\`}
                >
                  {st}
                </button>
              ))}
            </div>
          </div>
        ))
      )}
    </div>
  );
}

export function AdminQuickStatus({
  posts,
  onChangePostStatus,
}: {
  posts: CampusPost[];
  onChangePostStatus: (postId: string, status: PostStatus) => void;
}) {
  return (
    <div className="bg-white rounded-xl border border-slate-200 p-4 space-y-3">
      <h3 className="font-semibold text-slate-900 text-sm">Quick Post Status Controls</h3>
      <div className="space-y-2">
        {posts.map((p) => (
          <div
            key={p.id}
            className="flex flex-col sm:flex-row sm:items-center justify-between gap-2 py-2 border-b border-slate-100 last:border-none"
          >
            <div>
              <div className="text-sm font-medium text-slate-800">{p.title}</div>
              <div className="text-xs text-slate-500">
                {p.category} • {p.zone}
              </div>
            </div>
            <select
              value={p.status}
              onChange={(e) => onChangePostStatus(p.id, e.target.value as PostStatus)}
              className="px-2.5 py-1.5 text-xs rounded-lg border border-slate-200 bg-white font-medium"
            >
              <option value="Open">Open</option>
              <option value="Claimed">Claimed</option>
              <option value="Escalated">Escalated</option>
              <option value="Resolved">Resolved</option>
            </select>
          </div>
        ))}
      </div>
    </div>
  );
}

export function LocationTracker({
  activeZone,
  onSelectZone,
}: {
  activeZone: string;
  onSelectZone: (zone: string) => void;
}) {
  return (
    <div className="bg-white rounded-xl border border-slate-200 p-4 space-y-3">
      <div className="flex items-center justify-between">
        <span className="text-xs font-bold uppercase tracking-wider text-slate-500 flex items-center gap-1.5">
          <MapPin className="w-3.5 h-3.5 text-indigo-600" />
          Current Campus Zone
        </span>
        <span className="inline-flex items-center gap-1 text-xs text-emerald-700 font-medium">
          <Compass className="w-3.5 h-3.5" /> Active
        </span>
      </div>

      <select
        value={activeZone}
        onChange={(e) => onSelectZone(e.target.value)}
        className="w-full px-3 py-2 text-sm rounded-lg border border-slate-200 bg-slate-50 font-medium text-slate-800"
      >
        {CAMPUS_ZONES.map((z) => (
          <option key={z} value={z}>
            {z}
          </option>
        ))}
      </select>
    </div>
  );
}

export function VoiceAssistant({ activeZone }: { activeZone: string }) {
  const [open, setOpen] = useState(false);
  const [listening, setListening] = useState(false);
  const [input, setInput] = useState("");
  const [messages, setMessages] = useState<{ sender: "user" | "bot"; text: string }[]>([
    {
      sender: "bot",
      text: \`Hi! I'm your Smart Campus Assistant. Ask about lost items, claims, or escalations around \${activeZone}.\`,
    },
  ]);

  const askAssistant = async (text: string) => {
    const trimmed = text.trim();
    if (!trimmed) return;
    setMessages((prev) => [...prev, { sender: "user", text: trimmed }]);
    setInput("");

    try {
      const res = await fetch("/api/assistant", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ query: trimmed, zone: activeZone }),
      });
      const data = await res.json();
      setMessages((prev) => [...prev, { sender: "bot", text: data.reply }]);
    } catch {
      setMessages((prev) => [
        ...prev,
        {
          sender: "bot",
          text: \`You can report issues or check escalations in \${activeZone} directly from the top menu.\`,
        },
      ]);
    }
  };

  const toggleVoice = () => {
    const SpeechRecognitionCtor =
      typeof window !== "undefined"
        ? (
            window as unknown as {
              webkitSpeechRecognition?: new () => {
                start: () => void;
                onresult: (e: { results: ArrayLike<ArrayLike<{ transcript: string }>> }) => void;
                onend: () => void;
              };
            }
          ).webkitSpeechRecognition
        : undefined;

    if (!SpeechRecognitionCtor) {
      setMessages((prev) => [
        ...prev,
        { sender: "bot", text: "Voice input isn't supported in this browser—type your question below instead." },
      ]);
      return;
    }

    const recognition = new SpeechRecognitionCtor();
    setListening(true);
    recognition.onresult = (event) => {
      const spoken = event.results[0]?.[0]?.transcript || "";
      if (spoken) askAssistant(spoken);
    };
    recognition.onend = () => setListening(false);
    recognition.start();
  };

  return (
    <div className="fixed bottom-5 right-5 z-40">
      {!open ? (
        <button
          onClick={() => setOpen(true)}
          className="px-4 py-3 rounded-full bg-indigo-600 text-white shadow-lg hover:bg-indigo-700 flex items-center gap-2 text-sm font-semibold"
        >
          <Bell className="w-4 h-4" />
          Campus Assistant
        </button>
      ) : (
        <div className="w-80 sm:w-96 bg-white rounded-2xl shadow-2xl border border-slate-200 overflow-hidden flex flex-col">
          <div className="px-4 py-3 bg-indigo-600 text-white flex items-center justify-between">
            <div>
              <div className="text-sm font-semibold">Campus Assistant</div>
              <div className="text-[11px] text-indigo-100">Zone: {activeZone}</div>
            </div>
            <button onClick={() => setOpen(false)} className="p-1 rounded-lg hover:bg-indigo-500">
              <X className="w-4 h-4" />
            </button>
          </div>

          <div className="p-3 h-64 overflow-y-auto space-y-2.5 bg-slate-50">
            {messages.map((m, idx) => (
              <div
                key={idx}
                className={\`p-2.5 rounded-xl text-xs leading-relaxed
