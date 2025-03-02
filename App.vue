<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from "vue";
import { io } from "socket.io-client";

const socket = ref(null);
const messages = ref([]);
const newMessage = ref("");
const chatBox = ref(null);

// Establish socket connection
onMounted(() => {
    socket.value = io("http://localhost:3000", {
        reconnectionAttempts: 5, 
        reconnectionDelay: 3000 
    });

    socket.value.on("receiveMessage", (message) => {
        messages.value.push(message);
        scrollToBottom();
    });

    socket.value.on("connect_error", () => {
        console.error("WebSocket connection failed. Retrying...");
    });
});

// Cleanup on component unmount
onBeforeUnmount(() => {
    if (socket.value) {
        socket.value.disconnect();
    }
});

// Send message
const sendMessage = () => {
    if (newMessage.value.trim() !== "") {
        const messageObj = { user: "You", text: newMessage.value };
        messages.value.push(messageObj); 
        socket.value.emit("sendMessage", messageObj);
        newMessage.value = "";
        scrollToBottom();
    }
};

// Auto-scroll chat box
const scrollToBottom = () => {
    setTimeout(() => {
        if (chatBox.value) {
            chatBox.value.scrollTop = chatBox.value.scrollHeight;
        }
    }, 100);
};

// Watch messages and scroll to latest
watch(messages, scrollToBottom);
</script>

<template>
    <div class="chat-container">
        <h2>Real-Time Chat</h2>
        <div ref="chatBox" class="chat-box">
            <div v-for="(msg, index) in messages" :key="index" class="message" :class="{ 'self': msg.user === 'You' }">
                <strong>{{ msg.user }}:</strong> {{ msg.text }}
            </div>
        </div>
        <div class="input-box">
            <input v-model="newMessage" @keyup.enter="sendMessage" placeholder="Type a message..." />
            <button @click="sendMessage">Send</button>
        </div>
    </div>
</template>

<style scoped>
.chat-container {
    max-width: 400px;
    margin: 20px auto;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 10px;
    text-align: center;
    background: #ffffff;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.chat-box {
    height: 300px;
    overflow-y: auto;
    border: 1px solid #ccc;
    margin-bottom: 10px;
    padding: 10px;
    background: #f9f9f9;
    border-radius: 5px;
}

.message {
    background: #d1e7fd;
    padding: 8px;
    margin: 5px 0;
    border-radius: 5px;
    text-align: left;
}

.message.self {
    background: #a3e6a3;
    text-align: right;
}

.input-box {
    display: flex;
    gap: 5px;
}

input {
    flex: 1;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 5px;
    outline: none;
}

button {
    padding: 8px 15px;
    background: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background 0.3s;
}

button:hover {
    background: #0056b3;
}
</style>