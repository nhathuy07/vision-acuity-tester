<script context="module">
    //@ts-nocheck

export function getHistory() {
    let x = JSON.parse(localStorage.getItem("history"))
    if (x != null) {
        return x
    } else {
        return []
    }
}

export function clearHistory() {
        localStorage.setItem("history", JSON.stringify([]))
    }

</script>
