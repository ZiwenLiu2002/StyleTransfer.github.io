<template>
  <v-footer class="d-flex flex-column">
    <div class="d-flex w-100 align-center px-4" style="background-color: #97a5c0">
      <strong style="font-size: 15px; color: #fef0ef">
        Get connected with us on social networks!
      </strong>
      <v-spacer></v-spacer>
      <v-btn
        v-for="icon in icons"
        :key="icon"
        :icon="icon"
        class="mx-4"
        size="small"
        variant="plain"
        color="#ffffff"
      ></v-btn>
    </div>
    <div
      class="px-4 py-2 text-center w-100"
      style="background-color: #f9ece4; font-size: 24px; color: #12644e"
    >
      <strong style="color: #737c75">Style Transfer</strong>
    </div>
  </v-footer>

  <v-container fluid>
    <v-row>
      <v-col cols="12" sm="6">
        <v-card
          color="#f6e7eb"
          class="mx-auto"
          style="position: fixed; bottom: 20px; left: 1.2%; width: 82%; height: 14.5vh"
        >
          <v-card-item>
            <v-textarea
              v-model="text"
              bg-color="#f6e7f3"
              :loading="loading"
              density="compact"
              variant="solo-filled"
              hide-details
              single-line
              row-height="15"
              rows="3"
              :disabled="fileSelected"
              style="position: absolute; width: 89%; height: 10vh; margin: 0.4vh 0"
            >
              <template v-slot:label>
                <span style="color: #12644e; font-size: 18px">Your text here...</span>
              </template></v-textarea
            >
          </v-card-item>
          <v-btn
            color="#fcf5de"
            @click="onClick"
            :disabled="loading || (text.trim() === '' && !fileSelected) || !selectedStyle"
            :class="{
              'custom-disabled': loading || (text.trim() === '' && !fileSelected),
            }"
            :loading="loading"
            class="flex-grow-1"
            style="position: absolute; top: 5vh; right: 3vh; height: 5vh"
            density="compact"
          >
            <strong style="color: #737c75">Send</strong>
          </v-btn>
          <v-dialog max-width="500">
            <template v-slot:activator="{ props: activatorProps }">
              <v-btn
                style="
                  position: absolute;
                  right: 0.5%;
                  top: 5px;
                  width: 18px;
                  height: 18px;
                "
                v-bind="activatorProps"
                icon
                color="#f9ece4"
              >
                <v-icon color="#9DA8AF" style="font-size: 14px">mdi-help</v-icon>
              </v-btn>
            </template>

            <template v-slot:default="{ isActive }">
              <v-card color="#fafcfd">
                <v-card-title
                  style="
                    color: #4e6e62;
                    justify-content: center;
                    text-align: center;
                    padding-top: 25px;
                  "
                >
                  <strong>How to Use Style Transfer</strong>
                </v-card-title>
                <v-card-text style="color: #4e6e62">
                  <strong>1. Select a Style</strong><br />
                  Start by selecting a style from the
                  <strong>Styles</strong> dropdown list. You can choose from predefined
                  options or you can create a custom style by typing directly into the
                  selection box. <br /><br />

                  <strong>2. Input Text or Upload a File</strong><br />
                  After selecting a style, you can either: Type your text into the
                  provided text box or Upload a file by clicking on the file input. Note
                  that only
                  <em>.txt</em>
                  files are supported.

                  <br /><br />

                  <strong>3. Submit Your Request</strong><br />
                  Once you've selected a style and provided input (either by typing text
                  or uploading a file), click the
                  <strong>Send</strong> button to generate the style-transferred output.
                  <br /><br />

                  <strong>Tips:</strong><br />
                  The <strong>Send</strong> button will be enabled only after both a style
                  is selected and text or a valid file is provided.
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>

                  <v-btn
                    style="color: #4e6e62"
                    text="Close"
                    @click="isActive.value = false"
                    variant="elevated"
                    bg-color="#4e6e62"
                  ></v-btn>
                </v-card-actions>
              </v-card>
            </template>
          </v-dialog>
        </v-card>
      </v-col>
    </v-row>
  </v-container>

  <v-container>
    <v-row>
      <v-col cols="12">
        <div class="chat-container">
          <div v-for="(message, index) in messages" :key="index" class="chat-bubble">
            {{ message }}
          </div>
        </div>
      </v-col>
    </v-row>
  </v-container>

  <div class="chat-container">
    <div
      v-for="(message, index) in messages"
      :key="index"
      :class="['chat-bubble', message.isUser ? 'user' : 'reply']"
    >
      {{ message.text }}
    </div>
  </div>

  <div>
    <v-file-input
      ref="fileInput"
      style="position: fixed; right: 1.2%; bottom: 1.6vh; width: 14.5%"
      density="compact"
      clearable
      variant="solo"
      accept=".txt, text/plain"
      @change="validateFile"
    >
      <template v-slot:label>
        <span style="color: #12644e; font-size: 15px">Your file here...</span>
      </template></v-file-input
    >
  </div>

  <div class="container">
    <div class="content"></div>
    <v-combobox
      v-model="selectedStyle"
      style="position: fixed; bottom: calc(15.5vh - 7.5vh); right: 1.2%; width: 14%"
      density="compact"
      clearable
      label="Styles"
      :items="['bible', 'poetry', 'polite', 'Twitter', 'formal', 'negative']"
      variant="solo-filled"
    ></v-combobox>
  </div>
</template>

<script>
export default {
  data() {
    return {
      text: "",
      loading: false,
      icons: ["mdi-facebook", "mdi-twitter", "mdi-linkedin", "mdi-instagram"],
      messages: [],
      selectedStyle: null,
      fileSelected: false,
    };
  },
  methods: {
    validateFile(event) {
      const file = event.target.files[0];
      if (file && file.type !== "text/plain") {
        alert("Only TXT files are allowed!");
        event.target.value = ""; // Clear input
        this.fileSelected = false; // Reset file selection status
      } else if (file) {
        this.fileSelected = true; // File selection valid
      }
    },

    onClick() {
      this.loading = true; // Set loading status to true

      // File selection case
      if (this.fileSelected) {
        const fileInput = this.$refs.fileInput;
        const file = fileInput.files[0];
        const reader = new FileReader();

        reader.onload = (e) => {
          const content = e.target.result;
          this.messages.push({ text: content, isUser: true }); // Add file content to chat bubble
          fileInput.value = ""; // Clear file selection
          this.fileSelected = false; // Reset file selection status
          this.loading = false; // End loading status
          this.addEmptyBubble();
        };

        reader.readAsText(file); // Read file content
      }
      // Text input case
      else if (this.text.trim() !== "") {
        setTimeout(() => {
          this.messages.push({ text: this.text.trim(), isUser: true }); // Add text to chat bubble
          this.text = ""; // Clear text area
          this.loading = false; // End loading status
          this.addEmptyBubble();
        }, 500); // Delay for 500 milliseconds
      } else {
        this.loading = false; // End loading status if no input
      }
    },
    addEmptyBubble() {
      // Simulate a delay before adding an automated reply
      setTimeout(() => {
        this.messages.push({ text: "This is an automated reply.", isUser: false });
      }, 500);
    },
  },
};
</script>

<style scoped>
.chat-container {
  top: 110px;
  bottom: calc(15.5vh + 20px);
  width: 97.7%;
  overflow-y: auto;
  padding: 10px;
  background-color: #f8f8f8;
  border-radius: 10px;
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
}

.chat-bubble {
  background-color: #dde2e6;
  border-radius: 15px;
  padding: 10px 20px;
  margin: 10px 0;
  max-width: 55%;
  word-wrap: break-word;
  color: #4c5356;
  font-size: 16px;
  margin-left: auto;
}
/* User's message (right-aligned) */
.chat-bubble.user {
  margin-left: auto;
  background-color: #dde2e6;
}

/* Auto-reply message (left-aligned) */
.chat-bubble.reply {
  border-radius: 15px;
  padding: 10px 20px;
  margin: 10px 0;
  max-width: 55%;
  word-wrap: break-word;
  color: #4c5356;
  font-size: 16px;
  margin-right: auto;
  background-color: #c6d4d7;
}
.custom-disabled {
  background-color: #d1c3c3 !important; /* Change the background color */
  color: #f4f2f2 !important; /* Change the text color */
  cursor: not-allowed; /* Change cursor to indicate disabled */
}

/* Optional: You can also adjust hover effects */
.custom-disabled:hover {
  background-color: #f5f4f4 !important; /* Keep the same color on hover */
}
</style>
