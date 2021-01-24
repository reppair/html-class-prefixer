<template>
    <div class="container flex flex-col gap-4 py-8 h-screen">
        <div class="flex items-center space-x-8">
            <span class="font-semibold w-1/5">HTML Class Prefixer</span>

            <label class="ml-12 flex items-center space-x-4 w-full">
                <span>Input</span>
                <textarea
                    class="w-full bg-gray-100 h-8 rounded py-1 px-2 outline-none"
                    v-model="input"
                    placeholder="Paste markup"
                    autofocus
                />
            </label>

            <label class="ml-12 flex items-center space-x-4">
                <span>Prefix</span>
                <input ref="prefix" type="text" class="bg-gray-100 rounded py-1 px-2 outline-none" v-model="prefix" />
            </label>
        </div>

        <editor
            class="mt-2 bg-gray-50 rounded flex-auto p-4 overflow-x-auto"
            v-model="content"
            theme="pastel_on_dark"
            lang="html"
            @init="initAce"
            :options="editorOptions"
        />
        <span ref="tmp" v-html="tmp" class="hidden"></span>
    </div>
</template>

<script>
export default {
    name: 'Prefixer',

    components: {
        editor: require('vue2-ace-editor')
    },

    data: () => ({
        prefix: 'k-',
        input: '',
        content: '',
        originalContent: '',
        tmp: '',
        watch: true,
        editorOptions: {
            enableBasicAutocompletion: true,
            enableLiveAutocompletion: true,
            fontSize: 14,
            highlightActiveLine: true,
            enableSnippets: true,
            showLineNumbers: true,
            tabSize: 4,
            showPrintMargin: false,
            showGutter: true,
        },
    }),

    watch: {
        input(val) {
            if (this.watch)
                this.parse(val)
        },

        content(val, old) {
            if (this.watch && old === '')
                this.parse(val)
        },

        prefix(val, oldVal) {
            this.$nextTick(() => this.parse(this.content, oldVal))
        },
    },

    methods: {
        parse(val, oldPrefix = this.prefix) {
            this.tmp = val

            this.$nextTick(() => {
                this.$refs.tmp.querySelectorAll('*[class]').forEach(e => {
                    e.classList.forEach(cl => {
                        let original = cl
                        if (cl.includes(oldPrefix))
                            cl = cl.substring(oldPrefix.length)

                        let repl = cl.includes(':')
                            ? cl.split(':')[0] + ':' + this.prefix + cl.split(':')[1]
                            : this.prefix + cl;

                        e.classList.replace(original, repl)
                    })
                })

                this.watch = false
                setTimeout(() => this.content = this.$refs.tmp.innerHTML, 10)
                setTimeout(() => this.watch = true, 10)
                this.input = ''
                this.$refs.prefix.focus()
            })

        },

        initAce() {
            require('brace/ext/language_tools') //language extension prerequsite...
            require('brace/mode/html')
            require('brace/theme/pastel_on_dark')
            require('brace/snippets/html') //snippet
        },
    },
}
</script>
