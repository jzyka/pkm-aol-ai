<template>
  <v-app>
    <v-container style="max-height: 100vh; height: 100%">
      <div class="contain-all">
        <div class="header">
          <v-img src="@/assets/logo.png" contain class="logo"></v-img>
        </div>
        <div class="chat-container">
          <p class="banner" v-if="chatSession == null">
            What’s in your fridge?
          </p>
          <div class="parent-text" v-else>
            <div
              class="chat-chip"
              v-for="(message, index) in messages"
              :key="index"
            >
              <p v-html="formatMessage(message.text)"></p>
            </div>
          </div>
        </div>
        <div class="field-container">
          <v-text-field
            v-model="input"
            placeholder=" List your ingredients, I’ll suggest eco-friendly meals!"
            variant="solo"
            base-color="#818A91"
            bg-color="#212121"
            color="#787878"
            append-inner-icon="mdi-send"
            @click:append-inner="sendMessage"
          ></v-text-field>
        </div>
      </div>
    </v-container>
  </v-app>
</template>

<script>
import { GoogleGenerativeAI } from "@google/generative-ai";

export default {
  data() {
    return {
      input: "",
      messages: [],
      chatSession: null,
    };
  },
  methods: {
    async sendMessage() {
      if (!this.input.trim()) return;

      this.messages.push({ role: "user", text: this.input });
      const userInput = this.input;
      this.input = "";

      const apiKey = "AIzaSyCxRdnQkz2Lef12AxcUpnJIL0FdqrRHICw";
      const genAI = new GoogleGenerativeAI(apiKey);
      const model = genAI.getGenerativeModel({
        model: "tunedModels/finetunerecipes-gy2de66m677",
      });

      const generationConfig = {
        temperature: 1,
        topP: 0.95,
        topK: 64,
        maxOutputTokens: 8192,
        responseMimeType: "text/plain",
      };

      if (!this.chatSession) {
        this.chatSession = model.startChat({
          generationConfig,
          history: [],
        });
      }

      try {
        const result = await this.chatSession.sendMessage(userInput);
        this.messages.push({ role: "Bot", text: result.response.text() });
      } catch (error) {
        console.error("Error sending message:", error);
        this.messages.push({
          role: "Bot",
          text: "Error processing your request.",
        });
      }
    },
    formatMessage(text) {
      // Convert new lines to <br>
      text = text.replace(/\n/g, "<br>");

      // Convert bold markdown (**text**) to <strong>text</strong>
      text = text.replace(/\*\*(.*?)\*\*/g, "<strong>$1</strong>");

      // Convert italic markdown (*text*) to <em>text</em>
      text = text.replace(/\*(.*?)\*/g, "<em>$1</em>");

      // Convert markdown lists (- item) to <ul><li>item</li></ul>
      text = text.replace(/- (.*?)<br>/g, "<li>$1</li>");
      text = text.replace(/(<li>.*?<\/li>)+/g, "<ul>$&</ul>");

      return text;
    },
  },
};
</script>

<style lang="scss" scoped>
.contain-all {
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;

  .header {
    display: flex;
    justify-content: flex-start;

    .logo {
      height: 40px;
      max-width: 200px;
    }
  }

  .chat-container {
    height: 75%;
    overflow-y: scroll;
    display: flex;
    flex-direction: column;

    .banner {
      background: linear-gradient(90deg, #0198b0 0%, #00cf65 100%);
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      font-size: 48px;
      text-align: center;
      font-style: normal;
      font-weight: 400;
      line-height: normal;
      margin: auto;
    }

    .parent-text {
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      gap: 20px;
      padding-right: 20px;

      .chat-chip {
        max-width: 75%;
        width: fit-content;
        padding: 10px 25px;
        border-radius: 20px;

        &:nth-child(odd) {
          background-color: #787878;
          margin-left: auto;

          p {
            color: #ffffff;
          }
        }

        &:nth-child(even) {
          background-color: #212121;

          p {
            color: #ffffff;
          }
        }

        // p {
        //   text-wrap:
        // }
      }
    }
  }

  .field-container {
    // margin-bottom: 40px;
  }
}
</style>
