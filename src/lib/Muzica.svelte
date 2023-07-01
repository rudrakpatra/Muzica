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
    let debug=(el)=>{
        if(!isMobile()){
            el.onpointerdown = (e) => {
                type=e.type;
                target=e.currentTarget;
                timeStamp=e.timeStamp;
                changedTouches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
                touches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
            };
            el.onpointermove = (e) => {
                type=e.type;
                target=e.currentTarget;
                timeStamp=e.timeStamp;
                changedTouches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
                touches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
            };
            el.onpointerup = (e) => {
                type=e.type;
                target=e.currentTarget;
                timeStamp=e.timeStamp;
                changedTouches=[{identifier:0,clientX:e.clientX,clientY:e.clientY}];
                touches=[];
            };
        }else{
            el.ontouchstart = (e) => {
                type=e.type;
                target=e.currentTarget;
                timeStamp=e.timeStamp;
                changedTouches=Array.from(e.changedTouches);
                touches=Array.from(e.touches);
            };
            el.ontouchmove = (e) => {
                type=e.type;
                target=e.currentTarget;
                timeStamp=e.timeStamp;
                changedTouches=Array.from(e.changedTouches);
                touches=Array.from(e.touches);
            };
            el.ontouchend = (e) => {
                type=e.type;
                target=e.currentTarget;
                timeStamp=e.timeStamp;
                changedTouches=Array.from(e.changedTouches);
                touches=Array.from(e.touches);
            };
        }
        return {
            destroy(){
                el.ontouchstart=null;
                el.ontouchmove=null;
                el.ontouchend=null;
                el.onpointerdown=null;
                el.onpointermove=null;
                el.onpointerup=null;
            }
        }
    }
    //debug info
    let showDebug=true;
    let type="";
    let target="";
    let timeStamp=0;
    let changedTouches=[];
    let touches=[];

    let noteToKey=(note)=>{
        let octave=Math.floor(note/12);
        let noteName=["C","C#","D","D#","E","F","F#","G","G#","A","A#","B"][note%12];
        return noteName+octave;
    }
    let augmented=[48,52,56,60];
    let blues=[48,51,53,54,55,58,60];
    let pentatonic=[48,50,52,55,57,60];
    let minor=[48,50,51,53,55,56,58,60];
    let major=[48,50,52,53,55,57,59,60];
    let scales=[[],augmented,pentatonic,blues,minor,major];
    let noteOn=(note)=>{
        MIDI.noteOn(0, note, 120,0);
    }
    let noteOff=(note,delay)=>{
        MIDI.noteOff(0, note, delay);
    }
    let noteToTouchId=new Map();
    let touchIdToNote=new Map();
    let touchIDToScale=new Map();

    $:{
        if(type=="touchstart" || type=="pointerdown"){
            //find the touch whose identifier is not in touchIdToActiveNote
            let newTouch=touches.find(t=>!touchIdToNote.has(t.identifier))|| touches[0];
            let scale=scales[Math.floor(newTouch.clientY/innerHeight*scales.length)]
            let note=scale[Math.floor(newTouch.clientX/innerWidth*scale.length)];
            noteOn(note);
            touchIdToNote.set(newTouch.identifier,note);
            touchIDToScale.set(newTouch.identifier,scale);
            touchIdToNote=touchIdToNote;
        }
        if(type=="touchmove" || type=="pointermove"){
            //among the changed touches look for the ones that are in touchIdToActiveNote
            let activeTouches=touches.filter(t=>touchIdToNote.has(t.identifier));
            activeTouches.forEach(t=>{
                let prevScale=touchIDToScale.get(t.identifier);
                let note=touchIdToNote.get(t.identifier);
                let scale=scales[Math.floor(t.clientY/innerHeight*scales.length)]
                let newNote=scale[Math.floor(t.clientX/innerWidth*scale.length)];
                if(note!=newNote || scale!=prevScale){
                    noteOff(note);
                    noteOn(newNote);
                    touchIdToNote.set(t.identifier,newNote);
                    touchIDToScale.set(t.identifier,scale);
                    touchIdToNote=touchIdToNote;
                }
            })

        }
        if(type=="touchend" || type=="pointerup"){
            let note=touchIdToNote.get(changedTouches[0].identifier);
            noteOff(note);
            touchIdToNote.delete(changedTouches[0].identifier);
            touchIDToScale.delete(changedTouches[0].identifier);
            touchIdToNote=touchIdToNote;
        }
        console.log(mapEntriesToString(touchIdToNote));
    }
    $:{
        noteToTouchId= new Map(Array.from(touchIdToNote.entries()).map(([k,v])=>[v,k]));
    }
    function mapEntriesToString(entries) {
    return Array
        .from(entries, ([k, v]) => `\n  ${k}: ${v}`)
        .join(",") + "\n";
    }
</script>

<div class="debug" use:debug>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span class="action" on:click={_=>{showDebug=!showDebug}}>[{showDebug?"close debug panel":"open debug panel"}]</span><br/>
    {#if showDebug}
        <span>
            timeStamp={(timeStamp/1000).toFixed(2)}<br/>
            <!-- target={JSON.stringify(target)}<br/> -->
            type={JSON.stringify(type)}<br/>
            touches:
            {#each touches as touch}
                <span class:active={changedTouches.map(t=>t.identifier).includes(touch.identifier)}>
                    {touch.identifier}:({touch.clientX.toFixed(0)} , {touch.clientY.toFixed(0)})
                </span>,
            {/each}
            <br/>
            touchIdToNote:{mapEntriesToString(touchIdToNote)}<br/>
            noteToTouchId:{mapEntriesToString(noteToTouchId)}<br/>
        </span>
    {/if}
</div>

<div class="piano">
    {#each scales as scale}
        <div class="scale">
            {#each scale as note}
                <div class="key" class:active={noteToTouchId.has(note)}>
                    {noteToKey(note)}
                </div>
            {/each}
        </div>
    {/each}
</div>



<style>
    .action{
        color: gold;
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
        position:absolute;
        inset:0;
        display:flex;
        flex-direction:column;
    }
    .scale{
        /* uniform size */
        flex:1;
        display:flex;
        flex-direction:row;
    }
    .key{
        flex:1;
        display:flex;
        justify-content:center;
        align-items:center;
        font-size:1.5rem;
        font-weight:bold;
        color:rgba(255, 255, 255, 0.5);
        outline:1px solid rgba(255, 255, 255, 0.5);
        transition: transform 0.1s ease-out;
    }
    .key.active{
        color:gold;
        outline:1px solid gold;
        transform: scale(0.9);
    }
</style>