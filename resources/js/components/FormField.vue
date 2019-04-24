<template>
    <default-field :field="field" :errors="errors">
        <template slot="field">
            <div class="editorjs" :id="randID"></div>
        </template>
    </default-field>
</template>

<script>
    import {FormField, HandlesValidationErrors} from 'laravel-nova'

    const EditorJS = require('@editorjs/editorjs');
    const Paragraph = require('@editorjs/paragraph');
    const ImageTool = require('@editorjs/image');
    const CodeTool = require('@editorjs/code');
    const Header = require('@editorjs/header');
    const List = require('@editorjs/list');
    const LinkTool = require('@editorjs/link');
    const InlineCode = require('@editorjs/inline-code');

    export default {
        mixins: [FormField, HandlesValidationErrors],

        props: ['resourceName', 'resourceId', 'field'],
        data: function () {
            return {
                randID : ''
            }
        },
        methods: {
            /*
             * Set the initial, internal value for the field.
             */
            setInitialValue() {

                let self = this;
                let currentContent = (self.field.value ? JSON.parse(self.field.value) : self.field.value);

                this.randID = this.generateRandId(); //unique id for this instance of the editor.  You may need multiple on the page

                var editor = new EditorJS({
                    /**
                     * Wrapper of Editor
                     */
                    holderId:  this.randID,
                    /**
                     * Tools list
                     */
                    tools: {
                        header: {
                            class: Header,
                            inlineToolbar: ['link'],
                            config: {
                                placeholder: 'Header'
                            },
                            shortcut: 'CMD+SHIFT+H'
                        },
                        list: {
                            class: List,
                            inlineToolbar: true,
                            shortcut: 'CMD+SHIFT+L'
                        },
                        code: {
                            class: CodeTool,
                            shortcut: 'CMD+SHIFT+C'
                        },
                        linkTool: {
                            class: LinkTool,
                            config: {
                                endpoint: self.field.fetchUrlEndpoint,
                            }
                        },
                        image: {
                            class: ImageTool,
                            config: {
                                endpoints: {
                                    byFile: self.field.uploadImageByFileEndpoint,
                                    byUrl: self.field.uploadImageByUrlEndpoint,
                                },
                                additionalRequestHeaders: {
                                    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').getAttribute('content')
                                }
                            }
                        },
                        inlineCode: {
                            class: InlineCode,
                            shortcut: 'CMD+SHIFT+M',
                        },
                    },
                    /**
                     * This Tool will be used as default
                     */
                    initialBlock: 'paragraph',

                    /**
                     * Initial Editor data
                     */
                    data: currentContent,
                    onReady: function () {

                    },
                    onChange: function () {
                        editor.save().then((savedData) => {
                            self.handleChange(savedData)
                        });
                    }
                });
            },
            generateRandId : function(){
                return `editorjs_${Math.random().toString(36).substr(2, 9)}`;
            },

            /**
             * Fill the given FormData object with the field's internal value.
             */
            fill(formData) {
                formData.append(this.field.attribute, this.value || '')
            },

            /**
             * Update the field's internal value.
             */
            handleChange(value) {
                this.value = JSON.stringify(value)
            },
        },
    }
</script>
