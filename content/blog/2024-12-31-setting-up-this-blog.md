<!-- title: Setting Up This Blog -->
<!-- description: Generating a blog using only scripting and command line tools. And emacs, you can never get enough emacs. -->
<!-- updated_on: 2025-01-01 -->
Here we go.

## Software

When I decided to revive my blog the first thing I knew was I didn't want any content engines like Wordpress. I simply don't need the headache of maintaining public facing software and having every page render on demand. I wanted to follow the very successful prior art of [my old boss from his blog](https://www.tbray.org/ongoing/When/202x/2023/02/27/Twenty-ongoing-Years), where I could write my posts in emacs using markdown and with the push of a button generate a new static site. We're both on the same server running Apache, and Apache can serve static pages without even breathing hard.

Now, I'm not masochistic enough to write my own emacs major mode, but a little googling and [I found a little templating script](https://github.com/sunainapai/makesite) to do exactly what I wanted. It of course needed some customization, which the original author very much encouraged. I suppose I could have used an existing package like Jekyll, but I wanted something lighter weight.

So after a day of work, here I am. I can simply issue a `make publish` command, it will rebuild the site and push it out to the server. (I should create a `make staging` command too so I can preview things more easily before letting you all see it)

## Spell checking

We all make typos. So when editing in emacs how do I find spelling mistakes. Fortunately emacs has included `flyspell` since emacs 24. All one has to do it turn it on for the correct major mode and ensure you have hooks in place for right-click corrections. Add this fragment to your emacs initialization file and you'll start seeing little squiggles under words in text-mode and anything derived from it (which includes markdown mode).

```
; flyspell for spell checking
(dolist (hook '(text-mode-hook))
  (add-hook hook (lambda () (flyspell-mode 1))))

(eval-after-load "flyspell"
  '(progn
     (define-key flyspell-mouse-map [down-mouse-3] #'flyspell-correct-word)
     (define-key flyspell-mouse-map [mouse-3] #'undefined)))
```

## Markdown mode

This section is mainly for personal memory for whenever I upgrade my computer. You need to install emacs markdown mode, in Debian the package is `elpa-markdown-mode`.

![Emacs Flymode and Markdown modes](/images/2024/emacs-major-mode.png)

And voila! Highlighting for the markdown document as I write. I'll have to explore more emacs modes to add over time to make editing even easier. Emacs really is everything including the kitchen sink, and I wouldn't have my editor any other way.

## Future enhancements

When I reach a few dozen posts I'll probably have to rewrite the archiving system to break it into pages. And cleaning up the rendering engine, I split some functionality into two functions which in hindsight decreases maintainability. Also there's some code that could be unified from the original author's design. Maybe I'll get to it all, eventually.
