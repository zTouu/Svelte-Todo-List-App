<script>
    import { onMount } from "svelte";
    import { v4 as uuidv4 } from "uuid";

    import { items } from "../store";

    import TodoApi from "../TodoApi";
    import Item from "./item.svelte";
    import NewItem from "./Newitem.svelte";

    function handleNewItem(e) {
        $items = [
            {
                id: uuidv4(),
                text: e.detail,
                completed: false,
            },
            ...$items,
        ];

        TodoApi.save($items);
    }

    function handleUpdate(e) {
        const index = $items.findIndex(item => item.id === e.detail.id);
        $items[index] = e.detail;
        TodoApi.save($items);
    }

    function handleDelete(e) {
        $items = $items.filter(item => item.id !== e.detail);
        TodoApi.save($items);
    }

    let itemsSorted = [];

    $: {
        itemsSorted = [...$items];
        itemsSorted.sort((a, b) => {
            if (a.completed && b.completed) return 0;
            if (a.completed) return 1;
            if (b.completed) return -1;
        });
    }
    onMount(async() => {
        //fetch from API
        $items = await TodoApi.getAll();
    });
</script>

<style>
    .list{
        padding: 15px;
    }
    .list-status{
        margin: 0;
        text-align: center;
        color: #fff;
        font-weight: bold;
        font-size: 1.1em;
    }
</style>

<div class="list">
    <NewItem on:newitem={handleNewItem} />
    {#each itemsSorted as item (item)}
      <Item {...item} on:update={handleUpdate} on:delete={handleDelete} />
    {:else}
        <p class="list-status">No items exist</p>
    {/each}
</div>