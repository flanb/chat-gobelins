<script>
  import randomFonts from "$/functions/randomFonts"
  import { socket } from "$/stores.js"
  import Message from "./Message.svelte"

  let messagesContainer = null
  let messageHeight = 181

  let messages = []
  let currentUser = sessionStorage.getItem("id")

  const scrollToBottom = () => {
    setTimeout(() => {
      messagesContainer?.scrollTo(0, messagesContainer?.scrollHeight)
    }, 1)
  }

  // onMount(() => {
  //   messageHeight = messagesContainer.scrollHeight / messages.length
  // })

  socket.emit("getMessages")
  socket.on("messages", (data) => {
    messages = data
    scrollToBottom()
  })
  socket.on("message", (message) => {
    messages = [...messages, message]
    scrollToBottom()
  })

  const handleScroll = (event) => {
    // message who are in bottom of the screen is bigger than others
    const messageIndex =
      Math.floor(
        (event.target.scrollTop + event.target.clientHeight) / messageHeight
      ) - 1

    if (messageIndex >= 0) {
      messagesContainer.children[messageIndex - 3].style.transform = "scale(1)"
      messagesContainer.children[messageIndex - 2].style.transform =
        "scale(1.1)"
      messagesContainer.children[messageIndex - 1].style.transform =
        "scale(1.2)"
    }
    if (messagesContainer.children[messageIndex]) {
      messagesContainer.children[messageIndex].style.transform = "scale(1.3)"
    }
    if (messageIndex + 1 < messagesContainer.children.length) {
      messagesContainer.children[messageIndex + 1].style.transform = "scale(1)"
    }
  }
</script>

{#if messages.length}
  <ul bind:this={messagesContainer} on:scroll={handleScroll}>
    {#each messages as message}
      <Message
        content={message.value}
        avatarSrc={message.user.avatar}
        direction={currentUser === message.user.id ? "right" : "left"}
        color={message.user.color}
      />
    {/each}
  </ul>
{:else}
  <div class="blank">
    <p use:randomFonts>C’est pas qu’on s’ennuie mais bon...</p>
  </div>
{/if}

<style>
  ul {
    display: flex;
    flex-direction: column;
    overflow-y: auto;
    padding-top: 2rem;
    overflow-x: hidden;
  }
  .blank {
    text-align: center;
    font-size: 2rem;
    color: #e6e6e6;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 80%;
    margin: auto;
  }
</style>
