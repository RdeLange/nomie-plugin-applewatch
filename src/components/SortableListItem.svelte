<script>
	import { createEventDispatcher } from 'svelte';
	export let item;
	export let index;

	const dispatch = createEventDispatcher();


</script>

<style>
	
	.columnleft {
  float: left;
  width: 80%;
}
.columnright {
  float: right;
  width: 20%;
}

/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}
</style>

<div class="row">
	<div class="columnleft">
		<p style="cursor: grab">
			🟰 {item.emoji} {item.name}
		</p>
	</div>
	<div class="columnright">
		<p style="cursor: pointer" on:click={()=>{dispatch("delete",{index})}}>🗑</p>
	</div>
  </div> 

