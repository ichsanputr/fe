<template>
  <!-- component -->
  <div class="flex h-full overflow-hidden">
    <!-- Sidebar -->
    <div v-if="showContacts" class="absolute w-1/2 md:w-1/4 bg-white border-r border-b border-gray-300">
      <!-- Sidebar Header -->
      <header class="p-4 border-b border-gray-300 flex justify-between items-center bg-indigo-600 text-white">
        <h1 class="text-xl font-semibold">Hi {{ username }}👋</h1>
        <p @click="logout()" class="text-sm cursor-pointer font-medium underlined">Logout</p>
      </header>

      <!-- Contact List -->
      <div class="p-3">
        <div v-for="item in users" @click="changePairChat(item.chat_id)"
          class="flex items-center mb-4 cursor-pointer hover:bg-gray-100 p-2 rounded-md">
          <div class="w-12 h-12 bg-gray-300 rounded-full mr-3">
            <img src="https://placehold.co/200x/ffa8e4/ffffff.svg?text=ʕ•́ᴥ•̀ʔ&font=Lato" alt="User Avatar"
              class="w-12 h-12 rounded-full">
          </div>
          <div class="flex-1">
            <h2 class="text-lg font-semibold">{{ item.username }}</h2>
            <p class="text-gray-600">Hoorayy!!</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Main Chat Area -->
    <div class="flex-1">
      <!-- Chat Header -->
      <header class="bg-white p-4 text-gray-700 border-b-2 border-gray-100 flex justify-between">
        <h1 class="text-2xl font-semibold">{{ pairName }}</h1>
        <svg class="mt-1 mr-2 cursor-pointer" @click="showContacts = !showContacts" xmlns="http://www.w3.org/2000/svg"
          width="24" height="24" viewBox="0 0 24 24">
          <path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M5 17h14M5 12h14M5 7h14" />
        </svg>
      </header>

      <!-- Chat Messages -->
      <div class="overflow-y-auto px-4 pb-16">
        <div v-for="chat in currentPairChat">
          <!-- Incoming Message -->
          <div v-if="chat.is_me == 0" class="flex mb-4 cursor-pointer">
            <div class="w-9 h-9 rounded-full flex items-center justify-center mr-2">
              <img src="https://placehold.co/200x/ffa8e4/ffffff.svg?text=ʕ•́ᴥ•̀ʔ&font=Lato" alt="User Avatar"
                class="w-8 h-8 rounded-full">
            </div>
            <div class="flex max-w-96 text-white rounded-lg px-3 pt-1 gap-3 bg-indigo-500">
              <p>{{ chat.message }}</p>
            </div>
          </div>

          <!-- Outgoing Message -->
          <div v-else class="flex justify-end mb-4 cursor-pointer">
            <div class="flex max-w-96 bg-indigo-500 text-white rounded-lg p-3 gap-3">
              <p>{{ chat.message }}</p>
            </div>
            <div class="w-9 h-9 rounded-full flex items-center justify-center ml-2">
              <img src="https://placehold.co/200x/b7a8ff/ffffff.svg?text=ʕ•́ᴥ•̀ʔ&font=Lato" alt="My Avatar"
                class="w-8 h-8 rounded-full">
            </div>
          </div>
        </div>
      </div>

      <!-- Chat Input -->
      <footer class="bottom-0 fixed bg-white border-t border-gray-300 p-4 bottom-0" :class="showContacts ? 'w-full' : 'w-full'">
        <div class="flex items-center">
          <input v-model="message" type="text" placeholder="Type a message..."
            class="w-full p-2 rounded-md border border-gray-400 focus:outline-none focus:border-blue-500">
          <button @click="postMessage()" class="bg-indigo-500 text-white px-4 py-2 rounded-md ml-2">Send</button>
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      username: '',
      users: [],
      currentChat: '',
      currentPairChat: [],
      pairName: '',
      message: '',
      socket: '',
      showContacts: true
    }
  },
  async mounted() {
    if (!localStorage.getItem('token')) {
      this.$router.push('/login')
      return
    }

    this.socket = this.$nuxtSocket({
      name: 'main',
      channel: 'chat',
      extraHeaders: {
        token: localStorage.getItem('token')
      }
    })

    this.socket.on('notifyReloadChat', async (v) => {
      const { data } = await this.$axios.post(`${process.env.api}chat/pair`, {
        chat_id: v
      }, { headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` } });

      this.currentPairChat = data.data.chats
      this.pairName = data.data.pair_name
    })

    try {
      const data = await this.$axios.get(`${process.env.api}auth/profile`, { headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` } });
      this.username = data.data.data.user.username

      const resp = await this.$axios.get(`${process.env.api}chat/users`, { headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` } });
      this.users = resp.data.data
    } catch (err) {
      console.log(err)
    }
  },
  methods: {
    async changePairChat(id) {
      this.currentChat = id
      this.showContacts = false

      const { data } = await this.$axios.post(`${process.env.api}chat/pair`, {
        chat_id: this.currentChat
      }, { headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` } });

      this.currentPairChat = data.data.chats
      this.pairName = data.data.pair_name
    },
    async postMessage() {
      await this.$axios.post(`${process.env.api}chat/message`, {
        chat_id: this.currentChat,
        message: this.message
      }, { headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` } });

      const { data } = await this.$axios.post(`${process.env.api}chat/pair`, {
        chat_id: this.currentChat
      }, { headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` } });

      this.currentPairChat = data.data.chats
    },
    async logout() {
      localStorage.clear()
      this.$router.push('/login')
    }
  },
}
</script>