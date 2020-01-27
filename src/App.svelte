<script>
    import {writable, derived} from 'svelte/store'
    
    let input = null

    const todos = writable(JSON.parse(localStorage.getItem('store') || '[]'))
    const allComplete = derived(todos, ($todos) => {
        return $todos.every(x => x.complete)
    })
    const numIncomplete = derived(todos, ($todos) => {
        return $todos.filter(x => !x.complete).length
    })
    const numComplete = derived(todos, ($todos) => {
        return $todos.filter(x => x.complete).length
    })

    const commit = store => {
        localStorage.setItem('store', JSON.stringify(store))
    }

    const addTodo = item => {
        todos.update(x => {
            x.push({
                complete: false,
                description: item,
            })
            commit(x)
            return x
        })
    }

    const handleInput = e => {
        if (e.keyCode === 13) {
            addTodo(e.target.value)
            input = null
        }
    }

    const handleClick = index => {
        todos.update(x => {
            x[index].complete = !x[index].complete
            commit(x)
            return x
        })
    }

    const purge = () => {
        todos.update(x => {
            x = x.filter(y => !y.complete)
            commit(x)
            return x
        })
    }
</script>

<main>
    <h1>
        {#if $todos.length}
            {#if $allComplete}
                <strong>WooHoo!</strong> 0 items
            {:else}
                You have <strong>{$numIncomplete}</strong> 
                {#if $todos.length === 1}item{:else}items{/if} to do
            {/if}
        {:else}
            Let's get started!
        {/if}
    </h1>

    <section>
        <input 
            bind:value={input}
            on:keypress={handleInput} 
            placeholder="What do you need to do?" />
    </section>

    <section>
        <ul>
            {#each $todos as item, index}
                <li id={`todo-${index}`} class:complete={item.complete}>
                    <input 
                        type="checkbox" 
                        bind:checked={item.complete}
                        on:click={() => handleClick(index)} />
                    <label on:click={() => handleClick(index)}>
                        {item.description}
                    </label>
                </li>
            {:else}
                <em>No tasks entered</em>
            {/each}
        </ul>
    </section>

    <section>
        {#if $numComplete}
            <a href="#" on:click={purge}>Delete completed items</a>
        {/if}
    </section>
</main>

<style>
    :global(body) {
        background: #efefef;
    }

    main {
        text-align: center;
        padding: 1em;
        width: 860px;
        margin: 0 auto;
    }

    h1 {
        color: #ff3e00;
        text-transform: uppercase;
        font-size: 4em;
        font-weight: 100;
    }

    h1 strong {
        font-size: 1.25em;
    }

    ul {
        padding: 0;
        display: flex;
        flex-direction: column;
        align-items: start
    }

    li {
        list-style: none;
        flex: 1;
        display: flex;
        width: 100%;
        cursor: pointer;
    }

    label {
        cursor: pointer;
        vertical-align: middle;
    }

    li input {
        margin-right: 15px;
        margin-bottom: 0;
    }

    .complete {
        text-decoration: line-through;
    }
</style>