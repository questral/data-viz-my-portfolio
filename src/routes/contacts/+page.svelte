<!-- <nav>
    <ul>
        <li><a href="../index.html">home</a></li>
        <li><a href="../projects/index.html">projects</a></li>
        <li><a href="#">contacts</a></li>
        <li><a href="../resume/index.html">resume</a></li>
        <li><a href="https://github.com/questral" target="_blank">my git</a></li>
    </ul>
</nav> -->

<svelte:head>
	<title>contacts</title>
</svelte:head>

<h1>contacts</h1>

<form action="mailto:foo@example.com" method="get">
    <!-- <label>email<input name="email" type="email" value="h@email.com"></label> -->
    <label>subject<input name="subject" value="hello"></label>
    <label>body<textarea name="body">hello from this box</textarea></label>
    <button>submit</button>
</form>