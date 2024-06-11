<script>
    export let colorScale;

    colorScale.domain()
    colorScale.range()
</script>

<div class="legend">
    {#each colorScale.domain() as continent}
        <p> 
            <span style="background-color:{colorScale(continent)}"/>
            {continent} 
        </p>
    {/each}
</div>

<style>
.legend{
    display: flex;
    flex-direction: row;
    justify-content: center;
    flex-wrap: wrap;
    column-gap: 10px;
    row-gap: 5px;
    margin-bottom: 5px;
}

span{
    width: 9px;
    height: 9px;
    display: inline-block;
    border-radius: 50%;
    border: 1px solid rgba(0,0,0,0.4);
}

p{
    font-size: 14px;
    display: flex;
    text-transform: uppercase;
    align-items: center;
    column-gap: 3px;
}

</style>