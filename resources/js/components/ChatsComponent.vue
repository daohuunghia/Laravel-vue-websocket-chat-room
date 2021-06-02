<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <div class="card card-default">
                    <div class="card-header">Message</div>
                    <div class="card-body p-0">
                        <ul class="list-unstyled" style="height:300px; overflow-y:scroll"  v-chat-scroll>
                            <li v-for="(message, index) in messages" :key="index" class="p-2">
                               <strong>{{ message.user.name }}: </strong> {{ message.message }}
                            </li>
                        </ul>
                    </div>
                </div>
                <input type="text"
                       @keyup.enter="sendMessage"
                       @keydown="sendTypingEvent"
                       v-model="newMessage"
                       name="message"
                       class="form-control"
                       placeholder="Enter your message ..." />
                <span v-if="activeUser" class="text-muted">{{ activeUser.name }} is typing ...</span>
            </div>
            <div class="col-md-4">
                <div class="card card-default">
                    <div class="card card-header">Active Users</div>
                    <div class="card-body">
                        <ul>
                            <li v-for="(user, k) in users" :key="k" class="py-2">{{ user.name }} <span class="logged-in">●</span></li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ['user'],
    data () {
        return {
            messages: [],
            newMessage: '',
            users: [],
            activeUser: false,
            typingTimer: false
        }
    },
    created () {
        this.fetchMessages()
        Echo.join('chat')
            .here((users) => {
                console.log(users)
                this.users = users
            })
            .joining((user) => {
                console.log(user.name)
                this.users.push(user)
            })
            .leaving((user) => {
                console.log(user.name + ' Out')
                this.users = this.users.filter(u => u.id !== user.id)
            })
            .listen('SendMessageEvent', (e) => {
                this.messages.push(e.message)
            })
            .listenForWhisper('typing', (user) => {
                this.activeUser = user
                if (this.typingTimer) {
                    clearTimeout(this.typingTimer)
                }
                this.typingTimer = setTimeout(() => {
                    this.activeUser = false
                }, 3000)
            })
    },
    methods: {
        fetchMessages() {
            axios.get('messages').then(response => {
                this.messages = response.data
            })
        },
        sendMessage () {
            this.messages.push({
                user: this.user,
                message: this.newMessage
            })
            axios.post('messages', { message: this.newMessage })
            this.newMessage = ''
        },
        sendTypingEvent () {
            Echo.join('chat')
            .whisper('typing', this.user)
        }
    }
}
</script>
