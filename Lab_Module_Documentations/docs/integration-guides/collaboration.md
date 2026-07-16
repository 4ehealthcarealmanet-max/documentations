# 🤝 Integration: Healthcare Collaboration

This guide details how to implement and maintain the **Healthcare Collaboration** system, which enables Lab organizations to hire or collaborate with Doctors from the main HealthCare platform.

All collaboration data is hosted on the **HealthCare Core Backend** (default: `http://localhost:9000`).

---

## 📡 API Reference & Routes

| Feature | Method | Endpoint | Description |
| :--- | :--- | :--- | :--- |
| **Create Post** | `POST` | `/api/collaboration/posts/` | Publish a recruitment or medical camp requirement |
| **List Posts** | `GET` | `/api/collaboration/posts/` | Fetch all posts owned by this organization |
| **Close Post** | `POST` | `/api/collaboration/posts/{id}/close/` | Close the post to stop applications |
| **Send Message** | `POST` | `/api/collaboration/posts/{id}/message/` | Send a direct message to a doctor |
| **Conversations** | `GET` | `/api/collaboration/conversations/` | List conversation threads |
| **Messages** | `GET` | `/api/collaboration/messages/` | Fetch all messages within a conversation thread |

---

## 🐛 Critical Data Shape Gotchas & Defensive Coding

The collaboration database uses dynamic serializers. Relationship fields like `collaboration_post` or `doctor` may be returned as **nested JSON objects** or **flat UUID strings** depending on the endpoint query depth. 

Always implement defensive checks before accessing nested fields.

### 1. Handling `conversation.collaboration_post`
```typescript
// ✅ Safe check to match post IDs
const postId = typeof conversation.collaboration_post === 'object'
  ? (conversation.collaboration_post as any).id
  : conversation.collaboration_post;
```

### 2. Fetching Doctor Names safely
Sometimes the doctor profile name isn't populated on the user object. Fall back sequentially using this helper:
```typescript
export const getDoctorName = (conv: any): string => {
  const doctor = conv?.doctor;
  const lastMsg = conv?.last_message;
  return (
    doctor?.name ||
    doctor?.full_name ||
    (typeof lastMsg === 'object' && lastMsg?.sender === doctor?.id ? lastMsg?.sender_name : null) ||
    doctor?.username ||
    (typeof lastMsg === 'object' ? lastMsg?.sender_name : null) ||
    "Verified Doctor"
  );
};
```

---

## 💻 Service Layer (`collaboration.api.ts`)

Create a service file (`src/services/collaboration.api.ts`) with custom request interceptors that inject the patient or lab access token into headers:

```typescript
import axios from "axios";

const CONSULTATION_URL = process.env.NEXT_PUBLIC_CONSULTATION_URL || "http://localhost:9000";

const collabApi = axios.create({
  baseURL: CONSULTATION_URL,
  headers: { "Content-Type": "application/json" },
});

collabApi.interceptors.request.use((config) => {
  if (typeof window !== "undefined") {
    const token = localStorage.getItem("access");
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
  }
  return config;
});

export const collaborationService = {
  createPost: async (postData: any) => {
    const response = await collabApi.post("/api/collaboration/posts/", postData);
    return response.data;
  },
  getConversations: async () => {
    const response = await collabApi.get("/api/collaboration/conversations/");
    return response.data?.results || response.data;
  },
  sendMessage: async (postId: string, message: string, doctorId: string) => {
    const response = await collabApi.post(`/api/collaboration/posts/${postId}/message/`, {
      message,
      doctor_id: doctorId,
    });
    return response.data;
  }
};
```

---

## 🎨 Tabbed Collaboration Dashboard UI

The collaboration panel (`/lab/admin/collaboration`) is structured into a 3-tab layout.

```
┌────────────────────────────────────────────────────────┐
│  [ My Posts ]       [ Conversations ]     [ Alerts ]   │
├────────────────────────────────────────────────────────┤
│                                                        │
│   + New Requirement (Modal Trigger)                    │
│                                                        │
│   ┌──────────────────────────────────────────────┐     │
│   │ Blood Donation Camp - Sector 5               │     │
│   │ Urgency: HIGH | Status: ACTIVE               │     │
│   │ [ Close Post ]      [ View Conversations ]   │     │
│   └──────────────────────────────────────────────┘     │
└────────────────────────────────────────────────────────┘
```

1. **Tab 1: My Posts**:
   - Lists all posted camp or consulting roles.
   - Includes a detailed form inside a popup modal to create new posts.
2. **Tab 2: Conversations (Split Chat Layout)**:
   - **Left Panel**: Lists active threads showing Doctor Initial avatars, names, and a snippet of the last message.
   - **Right Panel**: A real-time message stream with standard blue/slate message bubbles and a message submission box.
3. **Tab 3: Notifications**:
   - Lists incoming collaboration alerts (e.g., `doctor_marked_available`).
   - Supports marking notifications as read individually or in bulk.
