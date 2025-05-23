<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  interface Message {
    id: number;
    text: string;
    time: string;
    sender: 'user' | 'bot';
  }

  let messages = writable<Message[]>([]);
  let inputText = '';

  let messageId = 0;

  function scrollToBottom() {
    const messageBody = document.getElementById("messageFormeight");
    if (messageBody) {
      messageBody.scrollTop = messageBody.scrollHeight;
    }
  }

  async function sendMessage() {
    const rawText = inputText.trim();
    if (!rawText) return;

    const now = new Date();
    const timeStr = now.getHours() + ":" + now.getMinutes().toString().padStart(2, '0');

    // Añadimos mensaje del usuario
    messages.update(msgs => [...msgs, { id: messageId++, text: rawText, time: timeStr, sender: 'user' }]);
    inputText = '';
    await tick(); // esperar actualización DOM
    scrollToBottom();

    // Pedimos respuesta al backend
    const res = await fetch("/get", {
      method: "POST",
      headers: { "Content-Type": "application/x-www-form-urlencoded" },
      body: new URLSearchParams({ msg: rawText })
    });

    const botText = await res.text();

    // Añadimos mensaje del bot
    messages.update(msgs => [...msgs, { id: messageId++, text: botText, time: timeStr, sender: 'bot' }]);
    await tick();
    scrollToBottom();
  }

  import { tick } from 'svelte';
</script>

<div class="container-fluid h-100">
  <div class="row justify-content-center h-100">
    <div class="col-md-8 col-xl-6 chat">
      <div class="card">
        <div class="card-header msg_head">
          <div class="d-flex bd-highlight">
            <div class="img_cont">
              <img
                src="/logo-uno.jpg"
                class="rounded-circle user_img"
                alt="Bot avatar"
              />
              <span class="online_icon"></span>
            </div>
            <div class="user_info">
              <span>ChatBot</span>
              <p>Preguntame algo</p>
            </div>
          </div>
        </div>

        <div id="messageFormeight" class="card-body msg_card_body">
          {#each $messages as message (message.id)}
            {#if message.sender === 'user'}
              <div class="d-flex justify-content-end mb-4">
                <div class="msg_container_send">
                  {@html message.text}
                  <span class="msg_time_send">{message.time}</span>
                </div>
                <div class="img_cont_msg">
                  <img src="https://i.ibb.co/d5b84Xw/Untitled-design.png" class="rounded-circle user_img_msg" alt="User avatar" />
                </div>
              </div>
            {:else}
              <div class="d-flex justify-content-start mb-4">
                <div class="img_cont_msg">
                  <img src="/logo-uno.jpg" class="rounded-circle user_img_msg" alt="Bot avatar" />
                </div>
                <div class="msg_container">
                  {@html message.text}
                  <span class="msg_time">{message.time}</span>
                </div>
              </div>
            {/if}
          {/each}
        </div>

        <div class="card-footer">
          <form
            id="messageArea"
            class="input-group"
            on:submit|preventDefault={sendMessage}
          >
            <input
              type="text"
              id="text"
              name="msg"
              placeholder="Type your message..."
              autocomplete="off"
              class="form-control type_msg"
              bind:value={inputText}
              required
            />
            <div class="input-group-append">
              <button
                type="submit"
                id="send"
                class="input-group-text send_btn"
                aria-label="Send message"
              >
                <i class="fas fa-location-arrow" aria-hidden="true"></i>
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>

<style>

  .chat {
    margin-top: auto;
    margin-bottom: auto;
  }
  .card {
    height: 500px;
    border-radius: 15px !important;
    background-color: rgba(0, 0, 0, 0.4) !important;
  }
  .card-header {
    border-radius: 15px 15px 0 0 !important;
    border-bottom: 0 !important;
  }
  .card-footer {
    border-radius: 0 0 15px 15px !important;
    border-top: 0 !important;
  }
  .msg_card_body {
    overflow-y: auto;
  }
  .type_msg {
    background-color: rgba(0, 0, 0, 0.3) !important;
    border: 0 !important;
    color: white !important;
    height: 60px !important;
    overflow-y: auto;
  }
  .type_msg:focus {
    box-shadow: none !important;
    outline: 0px !important;
  }
  .send_btn {
    border-radius: 0 15px 15px 0 !important;
    background-color: rgba(0, 0, 0, 0.3) !important;
    border: 0 !important;
    color: white !important;
    cursor: pointer;
  }
  .img_cont_msg {
    height: 40px;
    width: 40px;
  }
  .user_img_msg {
    height: 40px;
    width: 40px;
    border: 1.5px solid #f5f6fa;
  }
  .msg_container {
    margin-top: auto;
    margin-bottom: auto;
    margin-left: 10px;
    border-radius: 25px;
    background-color: rgb(82, 172, 255);
    padding: 10px;
    position: relative;
  }
  .msg_container_send {
    margin-top: auto;
    margin-bottom: auto;
    margin-right: 10px;
    border-radius: 25px;
    background-color: #58cc71;
    padding: 10px;
    position: relative;
  }
  .msg_time {
    position: absolute;
    left: 0;
    bottom: -15px;
    color: rgba(255, 255, 255, 0.5);
    font-size: 10px;
  }
  .msg_time_send {
    position: absolute;
    right: 0;
    bottom: -15px;
    color: rgba(255, 255, 255, 0.5);
    font-size: 10px;
  }
  .msg_head {
    position: relative;
  }
  .user_info {
    margin-top: auto;
    margin-bottom: auto;
    margin-left: 15px;
  }
  .user_info span {
    font-size: 20px;
    color: white;
  }
  .user_info p {
    font-size: 10px;
    color: rgba(255, 255, 255, 0.6);
  }
  .img_cont {
    position: relative;
    height: 70px;
    width: 70px;
  }
  .user_img {
    height: 70px;
    width: 70px;
    border: 1.5px solid #f5f6fa;
  }
  .online_icon {
    position: absolute;
    height: 15px;
    width: 15px;
    background-color: #4cd137;
    border-radius: 50%;
    bottom: 0.2em;
    right: 0.4em;
    border: 1.5px solid white;
  }

/*
  .offline {
    background-color: #c23616 !important;
  }
    */
</style>
