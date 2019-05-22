<template>
    <div class="ww-dropdown" @mouseover="enabledMenu = true" @mouseleave="enabledMenu = false" :class="{'open': editMode || forceOpen || focus }" :style="style">
        <!-- wwManager:start -->
        <wwOrangeButton class="ww-orange-button" v-if="editMode"></wwOrangeButton>
        <!-- wwManager:end -->
        <div class="dropdown-button-wrapper" :style="{'background-color': (enabledMenu ? activeColor:'')}">
            <wwObject tag="div" class="dropdown-button" :ww-object="wwObject.content.data.dropDownButton"></wwObject>
            <div class="dropdown-icon">
                <i class="dropdown-button-icon fas fa-chevron-down"></i>
            </div>
        </div>

        <div class="dropdown">
            <div class="triangle">
                <div class="triangle-after" :style="{'background-color': backgroundColor}"></div>
            </div>
            <div class="dropdown-list-wrapper" :style="{'background-color': backgroundColor}">
                <!-- add default background color with the possibility to change through popup  -->
                <!-- fix popup : remove extra options that are the button related -->
                <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="wwObject.content.data.list" class="dropdown-list" @ww-add="wwAdd($event)" @ww-remove="wwRemove($event)">
                    <wwObject class="dropdown-element" v-for="element in wwObject.content.data.list" :key="element" :ww-object="element"></wwObject>
                </wwLayoutColumn>
            </div>
        </div>
    </div>
</template>
 

<script>
/* 
triangle after change with a div 


*/
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
            forceOpen: false,
            enabledMenu: false,
            elementHover: false,
            activeColor: "#fafafa",
            elementColor: "#8f1afe"
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
        },
        editMode() {
            return this.wwObjectCtrl.getSectionCtrl().getEditMode() == 'CONTENT'
        },
        backgroundColor() {
            if (!this.wwObject || !this.wwObject.content)
                return "#ffffff"
            return (this.wwObject.content.data.background && this.wwObject.content.data.background.content) ? this.wwObject.content.data.background.content.data.backgroundColor : '#ffffff'
        }
    },
    watch: {
    },
    beforeDestroy() { },
    methods: {
        init() {
            this.wwObject.content.data = this.wwObject.content.data || {}


            if (!this.wwObject.content.data.background) {
                this.wwObject.content.data.background = wwLib.wwObject.getDefault({
                    type: 'ww-color',
                    data: {
                        backgroundColor: '#FFFFFF'
                    }
                });
                this.wwObjectCtrl.update(this.wwObject);

            }
            if (!this.wwObject.content.data.dropDownButton) {
                this.wwObject.content.data.dropDownButton = wwLib.wwObject.getDefault({
                    type: "ww-button",

                });
                this.wwObjectCtrl.update(this.wwObject);

            }

            if (_.isEmpty(this.wwObject.content.data.list)) {
                this.wwObject.content.data.list.push(
                    wwLib.wwObject.getDefault({
                        type: "ww-text",
                        data: {
                            text: {
                                fr: "place holder",
                                en: "place holder"
                            }
                        }
                    })
                )

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
            this.enabledMenu = !this.enabledMenu;
        },
        toggleColor() {
            this.enabledMenu = !this.enabledMenu;
        },
        setHoverColor(value) {
            this.elementHover = value

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
    .triangle {
        width: 20px;
        height: 20px;
        position: relative;
        overflow: hidden;
        float: right;
        right: 50%;
        top: 0;
        z-index: 10;
        transform: translate(50%, 0%);
        box-shadow: 0 16px 10px -17px rgba(0, 0, 0, 0.5);
    }
    .triangle-after {
        content: "";
        position: absolute;
        width: 15px;
        height: 15px;
        //background: #ffffff;
        transform: rotate(45deg);
        top: 13px;
        left: 2px;
        box-shadow: 0px 0px 6px -2px rgba(0, 0, 0, 0.5);
    }

    .dropdown-button-wrapper {
        width: 100%;
        height: 100%;
        padding: 15px;
        display: flex;
        align-items: center;
        border-radius: 15px;
        &.open,
        &:hover {
            background-color: #fafafa;
        }
        .dropdown-button {
            display: flex;
        }
        .dropdown-icon {
            display: flex;
            .dropdown-button-icon {
                margin-left: 5px;
                margin-right: 5px;
            }
            .rotate-icon {
                transform: rotate(180deg);
            }
        }
    }
    .dropdown {
        z-index: 10;
        min-width: 200px;
        position: absolute;
        top: calc(100% - 1px);
        opacity: 0;
        visibility: hidden;
        pointer-events: none;
        transition: opacity 0.3s ease;
        .dropdown-list-wrapper {
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            position: relative;
            .background {
                position: absolute;
                top: 0;
                left: 0;
                right: 0;
                bottom: 0;
            }
            .dropdown-list {
                z-index: 1;
                box-shadow: 0px 0px 3px 0px #bfbfbf;
                border-radius: 10px;
                padding: 10px;
                .dropdown-element {
                    width: 100%;
                    margin-bottom: 5px;
                    &.open,
                    &:hover {
                        background-color: #fafafa;
                        border-radius: 10px;
                    }
                }
            }
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

<style lang="scss">
.ww-dropdown {
    .dropdown {
        .dropdown-list {
            .dropdown-element {
                &.open,
                &:hover {
                    * {
                        color: #8f1afe !important;
                        border-color: #8f1afe !important;
                    }
                }
            }
        }
    }
}
</style>

