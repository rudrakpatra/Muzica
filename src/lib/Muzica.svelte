<script context="module">
    // @ts-ignore
    let MIDI=window.MIDI;
    MIDI.loadPlugin({
		soundfontUrl: "https://gleitz.github.io/midi-js-soundfonts/FatBoy/",
		instrument: "acoustic_grand_piano",
		onprogress: function(state, progress) {
			console.log(state, progress);
		},
		onsuccess: function() {
            MIDI.setVolume(0, 127);
            console.log("MIDI loaded",MIDI);
		}
	});
</script>

<script>
    let isMobile=()=>/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
    let touchHandlers=(el)=>{
        let handleTouchEventsEngage=(e)=>{
            type=e.type;
            target=e.currentTarget;
            timeStamp=e.timeStamp;
            changedTouches=Array.from(e.changedTouches);
            touches=Array.from(e.touches);
        }
        let handleTouchEventsRelease=(e)=>{
            type=e.type;
            target=e.currentTarget;
            timeStamp=e.timeStamp;
            changedTouches=[...touches];
            touches=[];
        }
        let handlePointerEventsEngage=(e)=>{
            type=e.type;
            target=e.currentTarget;
            timeStamp=e.timeStamp;
            changedTouches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
            touches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
        }
        let handlePointerEventsRelease=(e)=>{
            type=e.type;
            target=e.currentTarget;
            timeStamp=e.timeStamp;
            changedTouches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
            touches=[];
        }

        if(isMobile()){
            el.addEventListener("touchstart",handleTouchEventsEngage);
            el.addEventListener("touchmove",handleTouchEventsEngage);
            el.addEventListener("touchend",handleTouchEventsEngage);
            el.addEventListener("touchcancel",handleTouchEventsRelease);
            return {
                destroy(){
                    el.removeEventListener("touchstart",handleTouchEventsEngage);
                    el.removeEventListener("touchmove",handleTouchEventsEngage);
                    el.removeEventListener("touchend",handleTouchEventsEngage);
                    el.removeEventListener("touchcancel",handleTouchEventsRelease);
                }
            }
        }
        el.addEventListener("pointerdown",handlePointerEventsEngage);
        el.addEventListener("pointermove",handlePointerEventsEngage);
        el.addEventListener("pointerup",handlePointerEventsRelease);
        el.addEventListener("pointercancel",handlePointerEventsRelease);
        return {
            destroy(){
                el.removeEventListener("pointerdown",handlePointerEventsEngage);
                el.removeEventListener("pointermove",handlePointerEventsEngage);
                el.removeEventListener("pointerup",handlePointerEventsRelease);
                el.removeEventListener("pointercancel",handlePointerEventsRelease);
            }
        }
    }

    //debugging
    let showDebug=true;
    //debug info
    let type="";
    let target="";
    let timeStamp=0;

    //touch info
    let changedTouches=[];
    let touches=[];

    let active=new Map();
    export let layout=[];
    let rows=[[],...layout];
    console.log("rows",rows);

    //MIDI helpers
    let noteToKey=(note)=>{
        if(Array.isArray(note))
            return note.map(noteToKey).join(",");
        let octave=Math.floor(note/12);
        let noteName=["C","C#","D","D#","E","F","F#","G","G#","A","A#","B"][note%12];
        return noteName+octave;
    }
    let playNote=(note)=>{
        if (Array.isArray(note))
            MIDI.chordOn(0, note, 120, 0);
        else
            MIDI.noteOn(0, note, 127, 0);
    }
    let stopNote=(note)=>{
        if (Array.isArray(note))
            MIDI.chordOff(0, note, 0);
        else
            MIDI.noteOff(0, note, 0);
    }

    //layout helpers
    let getRowColAndNote=(x,y)=>{
        let rowNo=Math.floor(y/innerHeight*rows.length);
        let row=rows[rowNo];
        let colNo=Math.floor(x/innerWidth*row.length);
        return [rowNo,colNo,row[colNo]];
    }

    //event handlers
    let noteOn=(x,y,id)=>{
        // console.log("noteOn",x,y,id);
        let [rowNo,colNo,note]=getRowColAndNote(x,y);
        playNote(note);
        active.set(id,{rowNo,colNo,note});
        active=active;
    }
    let noteReplace=(x,y,id)=>{
        // console.log("noteReplace",x,y,id);
        let [rowNo,colNo,note]=getRowColAndNote(x,y);
        if(active.has(id)){
            let {rowNo:oldRowNo,colNo:oldColNo,note:oldNote}=active.get(id);
            if(rowNo!=oldRowNo || colNo!=oldColNo){
                stopNote(oldNote);
                playNote(note);
            }
        }else{
            playNote(note);
        }
        active.set(id,{rowNo,colNo,note});
        active=active;
    }
    let noteOff=(id)=>{
        // console.log("noteOff",id);
        let value=active.get(id);
        if(!value)return;
        let {rowNo,colNo,note}=value;
        stopNote(note);
        active.delete(id);
        active=active;
    }
    $:{
        if(type=="touchstart" || type=="pointerdown"){
            changedTouches.forEach(t=>noteOn(t.clientX,t.clientY,t.identifier));
        }
        if(type=="touchmove" || type=="pointermove"){
            changedTouches.forEach(t=>noteReplace(t.clientX,t.clientY,t.identifier));
        }
        if(type=="touchend" || type=="pointerup"){
            changedTouches.forEach(t=>noteOff(t.identifier));
        }
        if(type=="touchcancel" || type=="pointercancel"){
            changedTouches.forEach(t=>noteOff(t.identifier));
        }
    }
    $:activeNotesFlattened=Array.from(active.values()).map(v=>v.note).flat();
    $:console.log("activeNotesFlattened",activeNotesFlattened);
</script>
<div class="debug" use:touchHandlers>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    {#if showDebug}
        <span class="action" on:pointerdown={_=>{showDebug=false}}>[hide]</span><br/>
        <span>
            timeStamp={(timeStamp/1000).toFixed(2)}<br/>
            <!-- target={JSON.stringify(target)}<br/> -->
            type={JSON.stringify(type)}<br/>
            touches:
            {#each touches as touch}
                <span class:active={changedTouches.map(t=>t.identifier).includes(touch.identifier)}>
                    {touch.identifier}:({touch.clientX.toFixed(0)} , {touch.clientY.toFixed(0)})
                </span>,
            {/each}<br/>
            changedTouches:
            {#each changedTouches as touch}
                <span>
                    {touch.identifier}:({touch.clientX.toFixed(0)} , {touch.clientY.toFixed(0)})
                </span>,
            {/each}
            <br/>
            active:<br/>
            {#each Array.from(active.entries()) as entry}
                {JSON.stringify(entry)}<br/>
            {/each}
        </span>
    {:else}
        <span class="action" on:pointerdown={_=>{showDebug=true}}>[show]</span>
    {/if}
        
</div>

<div class="piano">
    {#each rows as row,i}
        <div class="row">
            {#each row as note,j}
                {#if Array.isArray(note)}
                    <div class="key chord" 
                    class:highlight={
                        //for all notes in the chord the note is in activeNotesFlattened
                        note.some(n=>activeNotesFlattened.some(a=>(a-n)%12==0))
                    }
                     class:active={
                        //the note is in activeNotesFlattened
                        note.every(n=>activeNotesFlattened.some(a=>a==n))
                    }
                    class:empty={
                        //the chord is empty
                        note.length==0
                    }>
                        {#each note as n}
                           <span>{noteToKey(n)}|{n}</span>
                        {/each}
                    </div>
                {:else}
                    <div class="key" 
                    class:highlight={
                        //the note has same letter as some note in activeNotesFlattened
                        activeNotesFlattened.some(n=>(n-note)%12==0)
                    }
                    class:active={
                       //the note is in activeNotesFlattened
                       activeNotesFlattened.some(n=>n==note)
                    }>
                        <span>{noteToKey(note)}|{note}</span>
                    </div>
                {/if}
            {/each}
        </div>
    {/each}
</div>



<style>
    .action{
        user-select: none;
        color: gold;
        padding:0.5rem;
        padding-right:4rem;
        font-size: 1rem;
    }
    .debug {
        /* display:none; */
        user-select: none;
        position: absolute;
        top:0;
        left:0;
        bottom:0;
        padding:1rem;
        width: calc(100% - 2rem);
        /* background: rgba(0, 163, 163, 0.2); */
        color: white;
        z-index: 1;
    }
    span.active{
        color:gold;
    }
    .piano{
        user-select: none;
        pointer-events: none;
        position:absolute;
        inset:0;
        display:flex;
        flex-direction:column;
    }
    .row{
        /* uniform size */
        flex:1;
        display:flex;
        flex-direction:row;
    }
    .key{
        flex:1 1 0px;
        min-width: 0;
        font-size:70%;
        display:flex;
        flex-direction: column;
        justify-content:center;
        align-items:center;
        gap:2px;
        
        font-weight:bold;
        color:rgba(255, 255, 255,0.8);
        outline:1px solid rgba(255, 255, 255 ,0.4);
        transition: transform 0.1s ease-out;
    }
    .key.highlight{
        color:hsla(43, 74%, 49%, 0.8);
        outline-color:hsla(43, 74%, 49%, 0.5);
        background-color: hsla(43, 74%, 49%, 0.1);
    }
    .key.active{
        color:hsl(51, 100%, 50%);
        outline-color:hsl(51, 100%, 50%);
        background-color: hsla(51, 100%, 50%, 0.2);
        transform: scale(0.9);
    }
    .chord.empty{
        color:rgba(255, 255, 255,0.2);
        outline-color:rgba(255, 255, 255 ,0.1);
        background-color: rgba(255, 255, 255 ,0.1);
    }
</style>