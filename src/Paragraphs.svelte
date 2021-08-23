<script>
 import { fly } from 'svelte/transition'
 import { onMount } from 'svelte'
 import { Story } from 'inkjs'

 let inkText = ''
 let story
 let choices = []
 let paragraphList = []
 let paragraphI = 0

 onMount(init)

 async function init() {
     const res = await fetch(`/gus.json`)
     const json = await res.json()
     story = new Story(json)

     story.ChoosePathString('talk')
     inkContinue()
 }


 // inkContinue gets all upcoming text from Ink,
 // up until the next choice
 function inkContinue() {
     while (story.canContinue) {
         let nextLine = story.Continue()

         if (nextLine.includes(': ')) {
             nextLine = nextLine.split(': ')[1]
         }

         inkText += nextLine
     }

     paragraphList = inkText.split('\n.\n')
     paragraphI = 0

     choices = story.currentChoices
 }

 function makeChoice(index) {
     inkText = ''
     paragraphList = []
     story.ChooseChoiceIndex(index)
     if (story.canContinue) {
         story.Continue()
     }
     inkContinue()
 }

 function advance() {
     const conversationOver = story && !story.canContinue && choices.length === 0 && paragraphI === paragraphList.length - 1

     if (conversationOver) {
         alert('done')

     }

     if (paragraphI < paragraphList.length - 1) {
        paragraphI = (paragraphI + 1)
     }
 }

 function open() {
    portrait = 'p-gus---3.png'
 }
 function close() {
     portrait = 'p-gus---2.png'
 }

 function primaryPress() {
    advance()
 }

 function primaryRelease() {
     // close()
 }

</script>

<main>
    {#key paragraphList}
        <pre style="padding: 3rem" in:fly="{{ y: 20, duration: 250 }}">{paragraphList.join('\n')}</pre>
    {/key}


        {#key choices}
            {#if choices && choices.length > 0}
                <div class="choice-list" in:fly="{{ y: 20, duration: 250, delay: 500 }}">
                    {#each choices as choice,i}
                        <button class="ab__option" on:click|preventDefault={() => makeChoice(i)}>{choice.text}</button>
                    {/each}
                </div>
            {/if}
        {/key}
</main>

<style>
 main {
     max-width: 450px;
     margin: 0 auto;

     position: relative;

     color: white;
 }

 .choice-list {
     display: flex;
     flex-direction: column;
     width: 100%;
     min-height: 150px;
     align-items: end;
 }

 .choice-list button {
     flex: 1;
     display: block;
     padding: 1.5rem;
     background: #A7ACE8;
     border-radius: 99rem;
     border: none;
     margin-bottom: 0.5rem;
 }

 .choice-list button:first-child {
     margin-top: 0.5rem;
 }

 .choice-list button:last-child {
     margin-bottom: none;
 }

</style>
