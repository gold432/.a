<script context="module">
    import * as api from '$lib/components/api.js';
    export async function load({session}) {
        let user = session.user
        if (!user){
            return {
                status: 302,
                redirect: 'enter'
            }
        }
        user = await api.get(`users/${user.id}`) || {}
        return {props: {user}}
    }
</script>
    
<script>
    export let user
    import { goto } from '$app/navigation';
    import {
        FluidForm,
        TextInput,
        Checkbox,
        Button,
        Column,
        Row,
        Tag
    } from 'carbon-components-svelte'
    import Input from '../lib/components/Input/Input.svelte'

    import { session } from '$app/stores'

    let username = user.username
    let visible = user.visible
    let token = user.token
    let tags = user.tags

    let usernameInvalid
    let usernameError
    let current
    let open
    let tag
    let ref

    $: if (username === '') {
        usernameInvalid=true
        usernameError='No username'
    } else {
        usernameInvalid=false
    }

    let clear = () => {
        open=false
        tags = []
    }

    let keydown = (e) => {
        switch(e.keyCode){
            case 13:
                if (e.ctrlKey){ edit()
                } else if (current==ref) addTag()
        }
    }

    let addTag = () => {
        if (tag != '' && !tags.includes(tag)){
            tags = [...tags, tag]
            open=true
            tag=''
        }
    }

    let delTag = (tag) => {
        tags = tags.filter(t => t != tag)
    }

    let checkUsername = async () => {
        if (username != user.username){
            usernameInvalid = await api.get(`check_username/${username}`).then(r => !r.res)
        }
    }

    let edit = async () => {
        let data = {
            username,
            visible,
            tags,
        } 
        let res = await api.put('users', data, token)
        if (res.id) {
            $session.user = res;
            goto('/')
        }
    }
</script>

<svelte:window on:keydown={keydown} />

<svelte:head>
    <title>Edit</title>
</svelte:head>

<Row noGutter>
    <Column>
        <Checkbox bind:checked={visible} labelText='Visible'/>
    </Column>
</Row>

<Row noGutter>
    <Column noGutter>
        <TextInput
            on:focus={() => {open=true; current=ref}}
            placeholder='Add tag'
            bind:value={tag}
            bind:ref
        />
    </Column>
</Row>

{#if open}
    <Row noGutter>
        <Column>
            <Tag
                on:click={clear}
                type='magenta'
            >
                Clear
            </Tag>
            {#each tags as tag}
                <Tag filter on:click={delTag(tag)}>{tag}</Tag>
            {/each}
        </Column>
    </Row>
{/if}
    
<Row noGutter>
    <Column>
        <FluidForm>
            <Input
                bind:invalidText={usernameError}
                bind:invalid={usernameInvalid}
                on:blur={checkUsername}
                bind:value={username}
                labelText="Username"
            />
        </FluidForm>
    </Column>
</Row>
    
<Row noGutter>
    <Column>
        <Button on:click={edit}>Edit</Button>
    </Column>
</Row>