<script context='module'> 
        export async function load({session}) {
            let user = session.user
        if(!user){
            return {
                status: 302,
                redirect: 'enter'
            }
        }
        return {props: {user}}
    }
</script>

<script>
    export let user
    import {
        Row,
        Link,
        Column
    } from 'carbon-components-svelte'
    import Tag from '../lib/components/Tag.svelte'
    import * as api from '$lib/components/api'
    import { goto } from '$app/navigation'
    import {
        roomTags
    } from '../stores.js'

    let rooms = []
    let page = 0
    let total = 0
    let got

    let go=async(room)=>{
        await api.put('join', {id: room.id}, user.token)
        goto(`room/${room.id}`)
    }

    let get = async function(){
        let tagString = JSON.stringify($roomTags)
        let url = `rooms?tags=${tagString}&visible=1&page=${page+1}`
        let res = await api.get(url, user.token)
        if(Array.isArray(res.items)){
            rooms = res.items
            // rooms.forEach(r=> console.log(r.name, r.s))
        }
        total = res.total
        got = true

    }
</script>

<svelte:head>
    <title>369</title>
</svelte:head>

<Tag on:change={get} bind:tags={$roomTags} />

{#each rooms as room}
    <br />
    <Row noGutter>
        <Column>
            <p class:unseen={room.unseen} class='item' href='' on:click={go(room)}>{room.name}</p>
        </Column>
    </Row>
{/each}

{#if got && total < 1}
    <Row noGutter>
        <Column>
            <p>There don't seem to be any results</p>        
        </Column>
    </Row>
{/if}

<style>
    .item {
        width: min-content;
        cursor: pointer; 
    }
    .item:hover {
        color: grey;
    }
    .unseen {
        font-weight: 600;
    }    
</style>