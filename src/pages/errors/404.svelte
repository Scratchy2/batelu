<script>
    let messages = [
        "How did we get here?",
        "I don't know about that one, mate.",
        "What?",
        "Just smile and nod...",
        "Try something else.",
        "???",
		"!!!",
		"?!?",
		"interrobang",
		"Wrong page!",
		"Turn back!",
		"hey this is actually a 403 page and you can't view this url.",
		"the server says nuh uh",
		"did you think you were getting somewhere with this?",
		"an iq too high?",
		"hop off bro"
    ]

    let message = messages[Math.floor(Math.random() * messages.length)]
</script>

<main class="container">
	<h1>Whoops!</h1>
	<p>Error 404 - Page not found!</p>
	<blockquote>{message}</blockquote>
</main>
