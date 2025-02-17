<template>
    <div id="MarkdownTooltip" class="noanim" ref="el" @keydown.esc="close" @mousedown="windowDrag">
        <span class="drag"><i class="fa-solid fa-grip-lines-vertical"></i></span>
        <el-tooltip v-for="act in actions" :key="act.name" :content="act.name">
            <span class="icon-wrapper" @mousedown="e => apply(e, act)">
                <i :class="act.icon"/>
            </span>
        </el-tooltip>
    </div>
</template>

<script lang="ts">
import {Options, Vue} from 'vue-class-component';
import {Prop, Ref} from "vue-property-decorator";

interface TooltipAction {
    name: string
    icon: string
    md: string
    // is: (text: string, start: number, end: number) => boolean
}

@Options({components: {}})
export default class MarkdownTooltip extends Vue
{
    actions: TooltipAction[] = [
        {name: '加粗',   icon: 'fa-solid fa-bold',          md: '**'},
        {name: '斜体',   icon: 'fa-solid fa-italic',        md: '__'},
        // {name: '下划线', icon: 'fa-solid fa-underline',     md: '--'},
        {name: '划掉',   icon: 'fa-solid fa-strikethrough', md: '~~'},
        {name: '代码',   icon: 'fa-solid fa-code',          md: '`'},
        {name: '黑幕',   icon: 'fa-solid fa-eye-slash',     md: '||'},
    ]

    @Ref() el!: HTMLElement
    @Prop({default: null}) initialPos?: {x: number, y: number}

    @Prop() textAreaId!: string
    textAreaEl: HTMLTextAreaElement
    selectedArea?: {start: number, end: number} = null

    mounted()
    {
        // Get text area element
        this.textAreaEl = document.getElementById(this.textAreaId) as HTMLTextAreaElement
        document.addEventListener('selectionchange', this.documentSelectionChange)
    }

    unmounted()
    {
        document.removeEventListener('selectionchange', this.documentSelectionChange)
    }

    /**
     * On document selection change
     *
     * This is necessary because there isn't a "deselect" event. One solution is to listen to blur,
     * focus, keydown, and mousedown events. It works most of the time, but when the user drags the
     * text, the mousedown event is called even though the text isn't deselected. So, we have to
     * listen to selection change event for the entire document instead.
     */
    documentSelectionChange(ev: UIEvent)
    {
        const active = document.activeElement
        const tel = this.textAreaEl

        // Textarea is not focused, then it's deselected
        if (active != tel) return this.close()

        // Selection is empty
        if (tel.selectionStart == tel.selectionEnd) return this.close()

        this.selectedArea = {start: tel.selectionStart, end: tel.selectionEnd}
        this.el.classList.add('show')
    }

    apply(e: UIEvent, act: TooltipAction)
    {
        e.preventDefault()
        let {start, end} = this.selectedArea
        let txt = this.textAreaEl.value
        let sel = txt.substring(start, end)

        // Change text while preserving history (TODO: Manually implement undo/redo
        let newTxt = txt.substring(0, start) + act.md + sel + act.md + txt.substring(end)
        document.execCommand('selectAll', false);
        const el = document.createElement('p');
        el.innerText = newTxt;
        document.execCommand('insertHTML', false, el.innerHTML);
        // this.textAreaEl.value = newTxt

        // Update selection range
        start += act.md.length
        end += act.md.length
        this.selectedArea = {start, end}
        this.textAreaEl.setSelectionRange(start, end)
    }

    close()
    {
        this.selectedArea = null
        this.el.classList.remove('show')
    }

    setPos(x: number, y: number): void
    {
        this.el.style.left = x + 'px'
        this.el.style.top = y + 'px'
    }

    /**
     * Window dragging
     */
    windowDrag(e: MouseEvent): void
    {
        // Only drag if it's dragging the root element, or the dragged element has 'drag' class
        function hasDrag(el: HTMLElement)
        {
            if (el.classList.contains('drag')) return true
            if (!el.parentElement) return false
            return hasDrag(el.parentElement)
        }
        if (!(e.target == this.el || hasDrag(e.target as HTMLElement))) return

        e.preventDefault()
        let lastX = e.clientX, lastY = e.clientY
        const mousemove = (e: MouseEvent) =>
        {
            const dx = lastX - e.clientX, dy = lastY - e.clientY
            lastX = e.clientX; lastY = e.clientY
            this.setPos(this.el.offsetLeft - dx, this.el.offsetTop - dy)
        }
        console.log(lastX, lastY)
        const mouseup = () => {document.removeEventListener('mouseup', mouseup); document.removeEventListener('mousemove', mousemove)}
        document.addEventListener('mouseup', mouseup)
        document.addEventListener('mousemove', mousemove)
    }
}
</script>

<style lang="sass">
@import src/css/global

#MarkdownTooltip:not(.show)
    opacity: 0
    pointer-events: none

#MarkdownTooltip
    position: absolute

    background: #fffcf9b5
    color: $color-text-light
    backdrop-filter: blur(10px)
    opacity: 0.9
    filter: drop-shadow(0 2px 5px rgba(166, 134, 89, 0.42))
    transition: opacity 0.5s ease

    border-radius: 10px
    padding: 5px

    white-space: nowrap

    .icon-wrapper + .icon-wrapper
        margin-left: 2px

    .icon-wrapper
        display: inline-flex
        align-items: center
        justify-content: center

        width: 30px
        height: 30px
        padding: 0
        border-radius: 8px

    .icon-wrapper:hover
        background: white

    .drag
        display: inline-block
        padding-left: 3px
        padding-right: 8px

</style>
