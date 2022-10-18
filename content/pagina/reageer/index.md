---
title: ✍️
page_class: simple-stretched
draft: false

---

<br/>

## Reageer op een recept of artikel

<br/>

Ik ben benieuwd naar je reactie.

<div style="max-width:600px">
<form action="https://385uplkgk8.execute-api.eu-central-1.amazonaws.com/main/message" method="POST">
<input type="hidden" name="_subject" value="Bericht verzonden vanaf pimsnel.nl">
<input type="hidden" name="_success_url" value="https://pimsnel.nl/pagina/bericht-verzonden/">
<input type="hidden" name="_fail_url" value="https://pimsnel.nl/contact/">
<input type="text" class="form-control mb-2" id="name" name="name" placeholder="Jouw naam">
<input type="email" class="form-control mb-2" id="email" name="email" placeholder="Jouw emailadres">
<textarea name="message" id="message" rows="6" class="form-control mb-2" placeholder="Bericht"></textarea>
<button type="submit" value="send" class="btn btn-block btn-primary rounded">Verzend</button>
</form>
</div>

<!-- vim: set spell spl=nl: -->
