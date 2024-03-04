<script>
    let dialog_0, dialog_1, dialog_2, dialog_3, dialog_4;
    let dialog_list = [dialog_0, dialog_1, dialog_2, dialog_3, dialog_4];

    $: {
        for(let i = 0; i < 5; i++) {
            if(dialog_list[i] && showList[i]) {
                dialog_list[i].showModal()
            }
        }
    }

    function swapModal(idx_1, idx_2) {
        dialog_list[idx_1].close();
        dialog_list[idx_2].show();
    }

</script>

<button on:click={() => (dialog_list[0].show())}> show modal </button>

<dialog
	bind:this={dialog_list[0]}
>
<p>Pick a movie you've seen out of</p>
<button on:click={() => swapModal(0,1)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[1]}
>
<p>Give it a rating!</p>
<button on:click={() => swapModal(1,2)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[2]}
>
<p>Let's see what everyone else thought.</p>
<button on:click={() => swapModal(2,3)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[3]}
>
<p>That's too many people! Let's take a sample.</p>
<button on:click={() => swapModal(3,4)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[4]}
>
<p>Not enough info. When we add them all to a histogram, we see...</p>
<button on:click={() => dialog_list[4].close()}>Close</button>
</dialog>