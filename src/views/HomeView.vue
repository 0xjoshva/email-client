<template>
  <main>
    <div class="modal-container">
      <div class="modal" v-if="createParticipantOpen == true">
        <input type="text" class="name" v-model="name" placeholder="John Doe" />
        <input
          type="email"
          class="email"
          v-model="email"
          placeholder="john@example.com"
        />
        <button class="add-participant" @click="addParticipant(name, email)">
          Add
        </button>
      </div>
    </div>

    <div
      class="container"
      :class="{
        moveDown: createParticipantOpen == true,
        moveUp: createParticipationOpen == false,
      }"
    >
      <div class="inner-container">
        <div
          class="participant-container"
          :class="{ focus: participantsFocused }"
          @mouseenter="participantsFocused = true"
          @mouseleave="participantsFocused = false"
        >
          <div class="participants">
            <div
              class="participant"
              v-for="participant in participants"
              :key="participant.id"
            >
              <p>{{ formatName(participant.name) }}</p>
              <button class="del" @click="deleteParticipant(participant.email)">
                ×
              </button>
            </div>
          </div>
          <button
            class="add-new"
            @click="createParticipantOpen = !createParticipantOpen"
          >
            +
          </button>
        </div>
        <input
          type="text"
          class="subject"
          placeholder="Subject"
          v-model="subject"
        />
        <textarea placeholder="Hi there," v-model="message"></textarea>
        <button @click="enhanceText()" class="enhance-btn">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="icon icon-tabler icon-tabler-wand"
            width="32"
            height="32"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="#000000"
            fill="none"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path stroke="none" d="M0 0h24v24H0z" fill="none" />
            <polyline points="6 21 21 6 18 3 3 18 6 21" />
            <line x1="15" y1="6" x2="18" y2="9" />
            <path
              d="M9 3a2 2 0 0 0 2 2a2 2 0 0 0 -2 2a2 2 0 0 0 -2 -2a2 2 0 0 0 2 -2"
            />
            <path
              d="M19 13a2 2 0 0 0 2 2a2 2 0 0 0 -2 2a2 2 0 0 0 -2 -2a2 2 0 0 0 2 -2"
            />
          </svg>
        </button>
        <button class="send-msg" @click="sendEmail()">Send message</button>
      </div>
    </div>
  </main>
</template>
<script>
import axios from "axios";
//import environment variables from .env
// import { GAPI_KEY, AI_API_KEY, GAPI_CLIENT_ID } from "../views/.env";

export default {
  name: "HomeView",
  data() {
    return {
      email: "",
      name: "",
      message: ``,
      subject: "",
      createParticipantOpen: false,
      participantsFocused: false,
      participants: [],
    };
  },
  computed: {
    participantEmails() {
      return this.participants.map((participant) => participant.email);
    },
  },
  methods: {
    formatName(name) {
      const [firstName, lastName] = name.split(" ");
      return `${firstName} ${lastName.charAt(0)}.`;
    },
    deleteParticipant(email) {
      const index = this.participants.findIndex(
        (participant) => participant.email === email
      );
      if (index !== -1) {
        this.participants.splice(index, 1);
        return true; // Return true if participant was successfully deleted
      }
      return false; // Return false if participant was not found
    },
    addParticipant(name, email) {
      const id = this.participants.length + 1; // Generate a unique id for the new participant
      const newParticipant = {
        id,
        name,
        email,
      };
      this.participants.push(newParticipant); // Push the new participant to the participants array
      this.createParticipantOpen = false;
      this.name = "";
      this.email = "";
    },
    sendEmail() {
      // Load the Gmail API client
      gapi.load("client", () => {
        // Initialize the Gmail API client
        gapi.client
          .init({
            apiKey: import.meta.env.VITE_GAPI_KEY,
            clientId: import.meta.env.VITE_CLIENT_ID,
            discoveryDocs: [
              "https://www.googleapis.com/discovery/v1/apis/gmail/v1/rest",
            ],
            scope: "https://www.googleapis.com/auth/gmail.compose",
          })
          .then(() => {
            // Build the email message
            const message = `To: ${this.participantEmails.join(
              ","
            )}\r\nSubject: ${this.subject}\r\n\r\n${this.message}`;

            // Encode the email message as base64
            const encodedMessage = btoa(message);

            // Send the email using the Gmail API
            gapi.client.gmail.users.messages
              .send({
                userId: "me",
                resource: {
                  raw: encodedMessage,
                },
              })
              .then(() => {
                // Reset the email form fields after successful email send
                this.participants = [];
                this.subject = "";
                this.message = "";
                // Show a success message to the user
                alert("Email sent successfully!");
              })
              .catch((error) => {
                console.error(`Failed to send email: ${error}`);
              });
          });
      });
    },
    resetParticipants() {
      this.participants = [];
    },
    async enhanceText() {
      try {
        const prompt = `Enhance the following text: ${this.message}`;
        const apiKey = import.meta.env.VITE_AI_API_KEY;
        const apiUrl =
          "https://api.openai.com/v1/engines/davinci-codex/completions";

        const headers = {
          "Content-Type": "application/json",
          Authorization: `Bearer ${apiKey}`,
        };

        const data = {
          prompt: prompt,
          max_tokens: 100,
        };

        const response = await axios.post(apiUrl, data, { headers });
        const enhancedText = response.data.choices[0].text;
        this.message = enhancedText;
      } catch (error) {
        console.error(`Failed to enhance text: ${error}`);
      }
    },
  },
};
</script>
<style>
@import url("https://fonts.cdnfonts.com/css/inter");
main {
  font-family: "Inter", sans-serif;
  background: #e8eaec;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  row-gap: 1rem;
}
.modal-container {
  width: 40%;
  display: flex;
  justify-content: start;
  height: 13rem;
}
.modal {
  height: 100%;
  width: 40%;
  background: #ffffff;
  box-shadow: 0px 6px 8px 2px #9e9e9e1c;
  border: 1px solid rgba(128, 128, 128, 0.432);
  border-radius: 16px;
  animation: pop-in ease 1s;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  row-gap: 1rem;
  padding: 1rem;
  z-index: 1;
  position: relative;
  transform: translate(0px, 50px);
}
.modal input {
  width: 100%;
  height: 44px;
  border-radius: 26px;
  border: 1px solid #d3d6d7;
  padding: 0.5rem;
  transition: ease all 0.4s;
  padding: 1rem;
}
.modal input:focus {
  outline: none;
  border: none;
  outline: none;
  border: 1px solid #2956f4;
  box-shadow: 0px 0px 0px 5px #dbe2fd;
}
.modal button {
  background: #2956f4;
  width: 100%;
  height: 44px;
  border-radius: 26px;
  outline: none;
  border: none;
  color: white;
  font-size: 18px;
  box-shadow: 0px 0px 0px 0px #dbe2fd00;
  transition: ease all 0.4s;
  cursor: pointer;
}
.modal button:focus {
  box-shadow: 0px 0px 0px 5px #dbe2fd;
  border: 1px solid #2956f4;
}
.container {
  background: #ffffff;
  width: 40%;
  height: fit-content;
  border: 1px solid rgba(128, 128, 128, 0.432);
  box-shadow: 0px 6px 8px 2px #9e9e9e21;
  border-radius: 16px;
  position: relative;
  z-index: 999;
}
.inner-container {
  padding: 2rem;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  row-gap: 1rem;
}
.participant-container {
  width: 100%;
  height: 44px;
  border-radius: 26px;
  border: 1px solid #d3d6d7;
  padding: 0.5rem;
  font-size: 16px;
  box-shadow: 0px 0px 0px 0px #dbe2fd00;
  transition: ease all 0.4s;
  display: flex;
  font-weight: 500;
  color: #2956f4;
  column-gap: 0.6rem;
  align-items: center;
  justify-content: space-between;
}
.participants {
  display: flex;
  column-gap: 0.6rem;
}
.participant {
  background: #eff3ff;
  width: fit-content;
  height: fit-content;
  padding-inline: 0.6rem;
  padding-block: 0.1rem;
  border-radius: 26px;
  display: flex;
  align-items: center;
  justify-content: center;
  column-gap: 0.5rem;
}
.del {
  line-height: 12px;
  width: 10px;
  height: fit-content;
  border: none;
  background: none;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #2956f4;
  font-size: 18px;
  cursor: pointer;
}
.add-new {
  border-radius: 50%;
  border: none;
  background: #2956f4;
  color: white;
  padding-inline: 0.5rem;
  transition: cubic-bezier(0.175, 0.885, 0.32, 1.275) all 0.2s;
  scale: 1.05;
  cursor: pointer;
}
.add-new:hover {
  scale: 1.1;
}
.add-new:focus {
  box-shadow: 0px 0px 0px 3px #dbe2fd;
}
.subject {
  width: 100%;
  height: 44px;
  border-radius: 26px;
  border: 1px solid #d3d6d7;
  padding: 1rem;
  font-size: 18px;
  box-shadow: 0px 0px 0px 0px #dbe2fd00;
  transition: ease all 0.4s;
}
.subject:focus {
  outline: none;
  border: 1px solid #2956f4;
  box-shadow: 0px 0px 0px 5px #dbe2fd;
}
::placeholder {
  color: #d3d6d7;
  font-weight: 400;
}
textarea {
  resize: none;
  width: 100%;
  height: 250px;
  border: 1px solid #d3d6d7;
  border-radius: 20px;
  padding-inline: 0.7rem;
  padding-block: 0.6rem;
  font-size: 18px;
  transition: ease all 0.4s;
}
textarea:focus {
  outline: none;
  border: 1px solid #2956f4;
  box-shadow: 0px 0px 0px 5px #dbe2fd;
}
.send-msg {
  background: #2956f4;
  width: 100%;
  height: 44px;
  border-radius: 26px;
  outline: none;
  border: none;
  color: white;
  font-size: 18px;
  box-shadow: 0px 0px 0px 0px #dbe2fd00;
  transition: ease all 0.4s;
  cursor: pointer;
}
.send-msg:focus {
  box-shadow: 0px 0px 0px 5px #dbe2fd;
  border: 1px solid #2956f4;
}
.focus {
  box-shadow: 0px 0px 0px 5px #dbe2fd !important;
  border: 1px solid #2956f4 !important;
}
@keyframes pop-in {
  from {
    transform: translate(0px, 300px);
    scale: 0.8;
  }
  to {
    transform: translate(0px, 50px);
  }
}
.moveDown {
  transition: ease all 1s;
  transform: translate(0px, 50px);
}
.enhance-btn {
  width: 40px;
  height: auto;
  border: none;
  outline: none;
  background: transparent;
  background: #dbe2fd;
  color: #2956f4;
  padding-inline: 0.3rem;
  padding-block: 0.3rem;
  border-radius: 50%;
  cursor: pointer;
  position: absolute;
  transform: translate(325px, 130px);
}
.enhance-btn svg {
  stroke: #2956f4;
}
</style>