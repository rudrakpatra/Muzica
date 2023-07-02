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
    // import { songs } from "./songs";
    let isMobile=()=>/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
    let touchHandlers=(el)=>{
        let handleTouchEvents=(e)=>{
            type=e.type;
            target=e.currentTarget;
            timeStamp=e.timeStamp;
            changedTouches=Array.from(e.changedTouches);
            touches=Array.from(e.touches);
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

        if(!isMobile()){
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
        el.addEventListener("touchstart",handleTouchEvents);
        el.addEventListener("touchmove",handleTouchEvents);
        el.addEventListener("touchend",handleTouchEvents);
        el.addEventListener("touchcancel",handleTouchEvents);
        return {
            destroy(){
                el.removeEventListener("touchstart",handleTouchEvents);
                el.removeEventListener("touchmove",handleTouchEvents);
                el.removeEventListener("touchend",handleTouchEvents);
                el.removeEventListener("touchcancel",handleTouchEvents);
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
    }
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
                <div class="key" 
                class:same={Array.from(active.values()).some(({note:n})=>Array.isArray(n)?n.includes(note):n==note)}
                class:active={Array.from(active.values()).some(({rowNo:r,colNo:c})=>r==i && c==j)}>
                    {noteToKey(note)}<br/>
                    {note}
                </div>
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
        font-size:80%;
        display:flex;
        justify-content:center;
        align-items:center;
        
        font-weight:bold;
        color:rgba(255, 255, 255);
        outline:1px solid rgba(255, 255, 255);
        opacity: 0.5;
        transition: transform 0.1s ease-out;
    }
    .key.same{
        color:goldenrod;
        outline:1px solid goldenrod;
        background-color: #222;
        opacity: 0.8;
    }
    .key.active{
        color:gold;
        outline:1px solid gold;
        background-color: #222;
        transform: scale(0.9);
        opacity: 1;
    }
</style>