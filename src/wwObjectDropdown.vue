<template>
    <div class="ww-dropdown" :class="{'open': forceOpen || focus}" :style="style">
        <!-- wwManager:start -->
        <wwOrangeButton class="ww-orange-button" v-if="wwObjectCtrl.getSectionCtrl().getEditMode() == 'CONTENT'"></wwOrangeButton>
        <!-- wwManager:end -->
        <wwObject :ww-object="wwObject.content.data.title" ww-inside-ww-object="ww-dropdown" :ww-not-editable="textNotEditable" ww-default-object-type="ww-text" ww-object-types-allowed="['ww-text']" :ww-no-section="wwAttrs.wwNoSection" :ww-no-link="wwAttrs.wwNoLink" ww-force-edit-mode="CONTENT"></wwObject>
        <div class="dropdown">
            <wwObject class="background" ww-category="background" :ww-object="wwObject.content.data.background" ww-inside-ww-object="ww-dropdown"></wwObject>
            <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="wwObject.content.data.list" class="dropdown-list" @ww-add="wwAdd($event)" @ww-remove="wwRemove($event)">
                <wwObject v-for="wwObj in wwObject.content.data.list" :key="wwObj.uniqueId" :ww-object="wwObj" ww-inside-ww-object="ww-dropdown"></wwObject>
            </wwLayoutColumn>
        </div>
    </div>
</template>
 

<script>

export default {
    name: "__COMPONENT_NAME__",
    props: {
        wwObjectCtrl: Object,
        wwAttrs: {
            type: Object,
            default: {}
        }
    },
    data() {
        return {
            textNotEditable: false,
            focus: false,
            forceOpen: false
        }
    },
    computed: {
        wwObject() {
            return this.wwObjectCtrl.get();
        },
        style() {
            let style = {};
            let wwObjectStyle = this.wwObject.content.data.style || {};
            style.justifyContent = wwObjectStyle.justify || 'center';
            return style;
        }
    },
    watch: {
    },
    beforeDestroy() { },
    methods: {
        init() {
            if (!this.wwObject.content.data.title || !this.wwObject.content.data.title.uniqueId) {

                let text = wwLib.wwObject.getDefault()
                text.content = wwLib.wwObject.getDefaultContent('ww-text')
                text.content.data.title = {
                    fr: 'Nouveau texte',
                    en: 'New text',
                }

                this.wwObject.content.data.title = text
                this.wwObjectCtrl.update(this.wwObject);
            }

            if (!this.wwObject.content.data.background || !this.wwObject.content.data.background.uniqueId) {

                let color = wwLib.wwObject.getDefault()
                color.content = wwLib.wwObject.getDefaultContent('ww-color');
                color.content.data.backgroundColor = '#FFFFFF';

                this.wwObject.content.data.background = color
                this.wwObjectCtrl.update(this.wwObject);
            }
        },

        /* wwManager:start */
        /*=============================================m_ÔÔ_m=============================================\
          EDIT
        \================================================================================================*/
        async edit() {

            wwLib.wwPopups.addStory('WWBUTTON_EDIT', {
                title: {
                    en: 'Edit button',
                    fr: 'Editer le bouton'
                },
                type: 'wwPopupEditWwObject',
                buttons: null,
                storyData: {
                    list: {
                        EDIT_LINK: {
                            separator: {
                                en: 'Link',
                                fr: 'Lien'
                            },
                            title: {
                                en: 'Change link',
                                fr: 'Changer le lien'
                            },
                            desc: {
                                en: 'External link, page link, ...',
                                fr: 'Lien externe, lien vers une page, ...'
                            },
                            icon: 'wwi wwi-link-external',
                            shortcut: 'l',
                            next: 'WWBUTTON_LINKS'
                        },
                        EDIT_STYLE: {
                            separator: {
                                en: 'Style',
                                fr: 'Style'
                            },
                            title: {
                                en: 'Change button style',
                                fr: 'Changer l\'apparence du bouton'
                            },
                            desc: {
                                en: 'Borders, colors, shadow, ...',
                                fr: 'Bordures, couleurs, ombres, ...'
                            },
                            icon: 'wwi wwi-edit-style',
                            shortcut: 's',
                            next: 'WWBUTTON_STYLE'
                        },
                        EDIT_JUSTIFY: {
                            title: {
                                en: 'Align',
                                fr: 'Alignement'
                            },
                            desc: {
                                en: 'Align left / right / center',
                                fr: 'Aligner à gauche / droite / centrer'
                            },
                            icon: 'wwi wwi-center',
                            shortcut: 'a',
                            next: 'WWBUTTON_ALIGN'
                        }
                    }
                }
            })


            wwLib.wwPopups.addStory('WWBUTTON_STYLE', {
                title: {
                    en: 'Button style',
                    fr: 'Style du bouton'
                },
                type: 'wwButtonPopupStyle',
                buttons: {
                    OK: {
                        text: {
                            en: 'Ok',
                            fr: 'Valider'
                        },
                        next: false
                    }
                }
            })

            wwLib.wwPopups.addStory('WWBUTTON_ALIGN', {
                title: {
                    en: 'Align',
                    fr: 'Alignement'
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: 'Align :',
                                fr: 'Alignement :'
                            },
                            type: 'select',
                            key: 'justify',
                            valueData: 'wwObject.content.data.style.justify',
                            options: {
                                type: 'text',
                                values: [
                                    {
                                        value: 'center',
                                        default: true,
                                        text: {
                                            en: 'Center',
                                            fr: 'Centrer'
                                        }
                                    },
                                    {
                                        value: 'flex-start',
                                        text: {
                                            en: 'Left',
                                            fr: 'Gauche'
                                        }
                                    },
                                    {
                                        value: 'flex-end',
                                        text: {
                                            en: 'Right',
                                            fr: 'Droite'
                                        }
                                    }

                                ]
                            }
                        }
                    ]
                },
                buttons: {
                    NEXT: {
                        text: {
                            en: 'Ok',
                            fr: 'Ok'
                        },
                        next: false
                    }
                }
            });

            let options = {
                firstPage: 'WWBUTTON_EDIT',
                data: {
                    wwObject: this.wwObject
                }
            }

            this.wwObjectCtrl.update(this.wwObject);
            console.log(this.wwObject)

            try {
                const result = await wwLib.wwPopups.open(options);

                /*=============================================m_ÔÔ_m=============================================\
                  STYLE
                \================================================================================================*/
                this.wwObject.content.data.style = this.wwObject.content.data.style || {};
                if (typeof (result.borderColor) != 'undefined') {
                    this.wwObject.content.data.style.borderColor = result.borderColor;
                }
                if (typeof (result.borderRadius) != 'undefined') {
                    this.wwObject.content.data.style.borderRadius = result.borderRadius;
                }
                if (typeof (result.borderStyle) != 'undefined') {
                    this.wwObject.content.data.style.borderStyle = result.borderStyle;
                }
                if (typeof (result.borderWidth) != 'undefined') {
                    this.wwObject.content.data.style.borderWidth = result.borderWidth;
                }
                if (typeof (result.boxShadow) != 'undefined') {
                    this.wwObject.content.data.style.boxShadow = result.boxShadow;
                }
                if (typeof (result.backgroundColor) != 'undefined') {
                    this.wwObject.content.data.style.backgroundColor = result.backgroundColor;
                }
                if (typeof (result.gradient) != 'undefined') {
                    this.wwObject.content.data.style.gradient = result.gradient;
                }
                if (typeof (result.gradientColor) != 'undefined') {
                    this.wwObject.content.data.style.backgroundColor = result.gradientColor;
                }
                if (typeof (result.padding) != 'undefined') {
                    this.wwObject.content.data.style.padding = result.padding;
                }
                if (typeof (result.justify) != 'undefined') {
                    this.wwObject.content.data.style.justify = result.justify;
                }


                this.wwObjectCtrl.update(this.wwObject);

                this.wwObjectCtrl.globalEdit(result);


            } catch (error) {
                console.log(error);
            }

        },
        setFocus(focusId) {
            this.focus = focusId == this.$parent._uid
        },
        wwAdd(options) {
            this.wwObject.content.data.list.splice(options.index, 0, options.wwObject);
            this.wwObjectCtrl.update(this.wwObject);
        },
        wwRemove(options) {
            this.wwObject.content.data.list.splice(options.index, 1);
            this.wwObjectCtrl.update(this.wwObject);
        },
        toggle() {
            this.forceOpen = !this.forceOpen;
        }
        /* wwManager:end */
    },
    mounted() {
        this.init();

        /* wwManager:start */
        wwLib.$on('wwFocus', this.setFocus);
        /* wwManager:end */

        this.$emit('ww-loaded', this);
    },
    beforeDestroy() {
        /* wwManager:start */
        wwLib.$off('wwFocus', this.setFocus);
        /* wwManager:end */
    }
};
</script>

<style scoped lang='scss'>
.ww-dropdown {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    cursor: pointer;

    &.open,
    &:hover {
        .dropdown {
            opacity: 1;
            visibility: visible;
            pointer-events: all;
        }
    }

    .dropdown {
        z-index: 10;
        position: relative;
        min-width: 100px;
        min-height: 50px;
        position: absolute;
        top: calc(100% - 1px);
        left: 50%;
        transform: translate(-50%, 0);
        opacity: 0;
        visibility: hidden;
        pointer-events: none;
        transition: opacity 0.3s ease;

        .background {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
        }

        .dropdown-list {
            z-index: 1;
        }
    }
}

/* wwManager:start */
.ww-orange-button {
    position: absolute;
    top: 0;
    left: 0;
    transform: translate(-50%, -50%);
    z-index: 1;
}
/* wwManager:end */
</style>
