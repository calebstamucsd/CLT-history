<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import StarRating from "@ernane/svelte-star-rating";

    const config = {
        readOnly: false,
        countStars: 5,
        range: {
            min: 0, 
            max: 5, 
            step: 0.5
        },
        score: 0.0,
        showScore: true,
        scoreFormat: function(){
            user_score = Math.max(((this.score * 2).toFixed(0) / 2), 0.5);
            return `(${user_score}/${this.countStars})`;
        },
        name: "",
        starConfig: {
            size: 30,
            fillColor: '#ffcf30',
            strokeColor: '#000000',
            unfilledColor: '#FFF',
            strokeUnfilledColor: '#000000'
        }
        }

    let dialog_0, dialog_1, dialog_2, dialog_3, dialog_4;
    let dialog_list = [dialog_0, dialog_1, dialog_2, dialog_3, dialog_4];
    let movie_choice, movie_data;
    let user_score;

    function swapModal(idx_1, idx_2) {
        dialog_list[idx_1].close();
        dialog_list[idx_2].show();
    }

    onMount(() => {
        d3.selectAll('img').on('click', function(event) {
            swapModal(0, 1)
            movie_choice = event.target.name;
            fetch('./src/components/assets/movie_ratings.json')
                .then(response => response.json())
                .then(data => {
                    movie_data = data[movie_choice];
                    console.log(movie_data);
                })
        })

    })
</script>

<button on:click={() => (dialog_list[0].show())}> show modal </button>

<dialog
	bind:this={dialog_list[0]}
>
<p>Pick a movie you've seen from the choices below. If you haven't seen any of them, that's okay - just play along!</p>
<div class='movie-container'>
    <div>
        The Lion King (1994)
        <br>
        <br>
        <img name="Lion King, The (1994)" src="./src/components/assets/images/the_lion_king.jpg" alt="The Lion King Movie Poster" width=140 height=210>
    </div>
    <div>
        Titanic (1997)
        <br>
        <br>
        <img name="Titanic (1997)" src="./src/components/assets/images/titanic.jpg" alt="Titanic Movie Poster" width=140 height=210>
    </div>
    <div>
        Spirited Away (2001)
        <br>
        <br>
        <img name="Spirited Away (Sen to Chihiro no kamikakushi) (2001)" src="./src/components/assets/images/spirited_away.jpg" alt="Spirited Away Movie Poster" width=140 height=210>
    </div>
    <div>
        Harry Potter and the Sorcerer's Stone (2001)
        <br>
        <img name="Harry Potter and the Sorcerer's Stone (a.k.a. Harry Potter and the Philosopher's Stone) (2001)" src="./src/components/assets/images/harry_potter.jpg" alt="Harry Potter and the Philosopher's Stone Movie Poster" width=140 height=210>
    </div>
    <div>
        The Avengers (2012)
        <br>
        <br>
        <img name="Avengers, The (2012)" src="./src/components/assets/images/the_avengers.jpg" alt="The Avengers Movie Poster" width=140 height=210>
    </div>
    <div>
        Parasite (2019)
        <br>
        <br>
        <img name="Parasite (2019)" src="./src/components/assets/images/parasite.jpg" alt="Parasite Movie Poster" width=140 height=210>
    </div>
</div>
<button on:click={() => swapModal(0,1)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[1]}
>
<p>Give it a rating!</p>
<StarRating {config}/>
<br>
<button on:click={() => swapModal(1,2)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[2]}
>
<p>Let's see what everyone else thought.</p>
<p>{user_score}</p>
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

<style>
    .movie-container {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
        grid-template-rows: 1fr 1fr;
        grid-gap: 10px
    }
</style>