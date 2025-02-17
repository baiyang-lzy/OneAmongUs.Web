<template>
    <div>
    <div id="home" :class="clicked ? 'clicked' : ''">
        <div class="introduction markdown-content" v-html="htmlTop" />

        <div id="profiles" class="unselectable" v-if="people">
            <div class="profile" v-for="p, i in people" :key="i">
                <div class="back"/>
                <transition name="fade" @after-leave="() => switchPage(p)">
                    <img :src="profileUrl(p)" draggable="false" alt="profile" class="front clickable"
                         v-if="clicked !== p.name"
                         @click="() => { if (!clicked) clicked = p.name }">
                </transition>
                <div class="sub-text font-custom">{{p.name}}</div>
                <div class="bookmark"/>
            </div>
            <div class="profile" v-if="showAdd">
                <div class="back add fbox-vcenter">+</div>
            </div>
        </div>

        <div class="introduction markdown-content" v-html="htmlBottom" />
    </div>
    </div>
</template>

<script lang="ts">
import {Options, Vue} from 'vue-class-component';
import htmlTop from "@/assets/home-top.md";
import htmlBottom from "@/assets/home-bottom.md";
import {PersonMeta} from "@/logic/data";
import {dataHost, replaceUrlVars} from "@/logic/config";
import urljoin from "url-join";

@Options({})
export default class Home extends Vue
{
    clicked = ''
    showAdd = false

    htmlTop = htmlTop
    htmlBottom = htmlBottom

    people: PersonMeta[] = null as never as PersonMeta[]

    created(): void
    {
        fetch(urljoin(dataHost, 'people-list.json'))
            .then(it => it.text())
            .then(it => {
                this.people = JSON.parse(it)
                console.log(it)
                console.log(this.people)
            })
    }

    switchPage(p: PersonMeta): void
    {
        console.log("Called, " + p.id)
        this.$router.push(`/profile/${p.id}`)
    }

    profileUrl(p: PersonMeta): string
    {
        return replaceUrlVars(p.profileUrl, p.id)
    }
}
</script>

<style lang="sass" scoped>
@import "../css/colors"

.introduction
    text-align: justify
    text-justify: inter-word
    margin: 10px min(5vw, 40px)

#profiles
    margin-top: 20px

// Profile picture alignment
.profile
    position: relative
    display: inline-block
    margin: 20px 20px 30px
    vertical-align: top

    .fade-enter-active, .fade-leave-active
        transition: all .25s ease !important

    .fade-enter, .fade-leave-to
        opacity: 0

    .front, .back
        border: 10px solid white
        outline: 2px solid $color-text-main
        height: 150px
        width: 150px
        transition: all .25s ease

    .back
        z-index: 2
        position: relative

    .front
        transform: rotate(10deg)
        position: absolute
        z-index: 4
        height: 150px
        top: 0
        left: 0

    // Hover animation
    .front:hover
        transform: rotate(2deg)

    .sub-text
        margin-top: 3px
        margin-left: 15px
        text-align: left
        position: relative
        z-index: 3

    .back.add
        outline: 2px dashed $color-text-main
        font-size: 75px
        color: gray
        background-color: #f1f1f1

    .bookmark
        border: 40px solid rgba(255, 189, 202, 0.49)
        //$border: pink
        //filter: drop-shadow(0 2px 0 $border) drop-shadow(2px 0 0 $border) drop-shadow(-2px 0 0 $border)
        border-bottom: 10px solid transparent
        // width:        100px
        width: 0
        left: 10px
        height: 10px

        position: absolute
        bottom: -15px
        z-index: 2

@media screen and (max-width: 440px)
    .profile
        .front, .back
            border: 5px solid white
            $len: 30vw
            height: $len
            width: $len
</style>
