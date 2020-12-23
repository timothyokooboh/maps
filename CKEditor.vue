<template>
  <div class="container">
    <div id="toolbar-container"></div>
    <div id="editor" @input="getValue"></div>
  </div>
</template>

<script>
/**
 * MAY BE USED TO SANITIZE THE HTML
 * 
 * import sanitizeHtml from "sanitize-html";
 */

/**
 * A div element is used in place of textarea because textarea cannot be passed
 * to the create() function of the editor class. More info can be found here https://ckeditor.com/docs/ckeditor5/latest/framework/guides/support/error-codes.html#error-editor-wrong-element
 * 
 * HOW TO USE THIS COMPONENT
 * 
 * Wherever this component is called, the "user-input" custom event must be listened to.
 * The value of the event is equal to the value entered by the user in the text editor.
 * 
 * E.G <CKEditor :user-input="userInput" />
 * 
 * methods: {
 *    userInput(value) {
 *      console.log(value)
 *   }
 * }
 * 
 * 
 * The server returns the submitted value as HTML (texts with open and close tags)
 * 
 * Therefore, use v-html to display the returned value on the webpage.
 * 
 * E.G Instead of <div> {{bio}} </div>
 *  use, <div v-html="bio"></div>
 * 
 * The "value" prop is used to initialize the content of the editor on page load.
*/

let DecoupledEditor;

if (process.browser) {
  DecoupledEditor = require('@ckeditor/ckeditor5-build-decoupled-document')
}

export default {
    props: ["value"],
    data() {
        return {
            text: ""
        }
    },
    watch: {
        value: {
            handler(newValue) {
                document.querySelector("#editor").innerHTML = newValue;
            }
        }
    },
    methods: {
        getValue() {
            let userInput = document.querySelector("#editor").innerHTML;
            this.text = userInput;
            this.$emit("user-input", this.text);
        }
    },
    mounted() {
        DecoupledEditor.create(document.querySelector('#editor'))
        .then((editor) => {
            const toolbarContainer = document.querySelector('#toolbar-container')
            toolbarContainer.appendChild(editor.ui.view.toolbar.element)
        })
        .catch((error) => {
        // eslint-disable-next-line no-console
        console.error(error)
        })
    }
}
</script>

<style lang="scss" scoped>

    .container {
        #editor {
            border: 2px solid #ddd;
            min-height: 70px;
            padding-left: 25px // MAKES LISTS TO BE RENDERED PROPERLY
        }
    }
</style>