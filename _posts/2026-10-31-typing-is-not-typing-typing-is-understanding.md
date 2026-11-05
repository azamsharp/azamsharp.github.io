# Typing Is Not Typing, Typing Is Understanding

When I was growing up in Pakistan, my teacher used to tell me to copy text from my textbook into my notebook. And by writing, I mean physically writing with a pen and paper (yes, paper that comes from trees).

I never understood the point of this exercise. The text was already printed in the book, so why did I have to write the same text in my notebook?

I eventually realized that writing the text helped me actively learn and understand the concepts better. I strongly believe the same is true for programming.

I am always surprised when developers reveal that they have not written a single line of code manually in the last 3 to 6 months. When I ask them how they know whether the code is high quality, they tell me that sometimes they review the generated code, and sometimes they don't even look at it.

> If you are working on small personal projects where the stakes are low and the blast radius is nonexistent, then there is nothing wrong with vibe coding your entire app.

Reviewing AI generated code is not the same as writing it yourself. When you write code, you constantly make decisions about design, architecture, and implementation. But when you review code that already works, there is a tendency to accept the existing solution instead of exploring better alternatives. Just because the code works does not mean it is well designed.

But I think the biggest disadvantage of letting AI write all the code for you is that you give up an opportunity to learn and understand the code. You choose short term gains (working code) at the expense of long term learning, potentially leading to what some might call brain rot.

Just like your body can deteriorate if you don't exercise, the same can happen to your brain. You need those moments of struggle when you make your brain work.

When I am working on projects, I tend to use AI as a reference, much like Google. But I think even that is causing me to miss out on learning opportunities because AI is very quick at providing answers and removes many of the hurdles involved in finding solutions myself.

Keep in mind that exploring all the different solutions that did not work can be just as important as finding the one that did. Using AI tools can remove that struggle, which in turn can lead to missed learning opportunities. Even if I struggle for 15 to 20 minutes, that experience can be far more valuable than getting the answer in mere seconds.

I am not suggesting that you write every single line of code by hand or stop using AI tools. I use AI myself, and I recognize the enormous productivity benefits it provides.

But I also believe that not every struggle needs to be eliminated. Sometimes, spending 20 minutes trying to solve a problem teaches you more than getting the correct answer in 20 seconds.

When I was growing up, I thought my teacher was wasting my time by asking me to copy text that was already printed in a book. Looking back, I realize that the goal was never to produce another copy of the text. The goal was to learn and understand it.

I think the same is true for programming.

**AI can write the code for you, but it cannot do the learning for you.**

<!-- Book Banner: SwiftUI Architecture Book -->
<div class="azam-book-banner" role="region" aria-label="SwiftUI Architecture Book Banner">
  <div class="azam-book-banner__inner">
    <div class="azam-book-banner__cover">
      <img
        src="https://azamsharp.com/images/swiftui-book-1.png"
        alt="SwiftUI Architecture book cover"
        loading="lazy"
      />
    </div>
    <div class="azam-book-banner__content">
      <p class="azam-book-banner__eyebrow">SwiftUI Architecture Book</p>
      <h3 class="azam-book-banner__title">Patterns and Practices for Building Scalable Applications</h3>
      <p class="azam-book-banner__subtitle">
        A practical guide to building SwiftUI apps that stay clean as they grow.
      </p>
      <div class="azam-book-banner__actions">
        <a class="azam-book-banner__button" href="https://azamsharp.school/swiftui-architecture-book.html" target="_blank" rel="noopener">
          Get the book
        </a>
      </div>
    </div>
  </div>
</div>

<style>
  .azam-book-banner {
    --bg1: #0b1220;
    --bg2: #111a2d;
    --text: rgba(255, 255, 255, 0.92);
    --muted: rgba(255, 255, 255, 0.74);
    --border: rgba(255, 255, 255, 0.12);
    --shadow: 0 18px 45px rgba(0, 0, 0, 0.28);
    --accent: #6ee7b7; /* tweak to match your brand */
    --accent2: #60a5fa;

    margin: 22px 0;
    color: var(--text);
    border: 1px solid var(--border);
    border-radius: 16px;
    overflow: hidden;
    background: radial-gradient(1200px 600px at 10% 0%, rgba(96, 165, 250, 0.22), transparent 60%),
                radial-gradient(900px 500px at 90% 30%, rgba(110, 231, 183, 0.18), transparent 60%),
                linear-gradient(135deg, var(--bg1), var(--bg2));
    box-shadow: var(--shadow);
  }

  .azam-book-banner__inner {
    display: grid;
    grid-template-columns: 132px 1fr;
    gap: 18px;
    padding: 18px;
    align-items: center;
  }

  .azam-book-banner__cover {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .azam-book-banner__cover img {
    width: 132px;
    height: auto;
    border-radius: 12px;
    border: 1px solid rgba(255,255,255,0.14);
    box-shadow: 0 14px 28px rgba(0,0,0,0.35);
    background: rgba(255,255,255,0.04);
  }

  .azam-book-banner__eyebrow {
    margin: 0 0 6px 0;
    font-size: 12px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--muted);
  }

  .azam-book-banner__title {
    margin: 0 0 8px 0;
    font-size: 18px;
    line-height: 1.25;
  }

  .azam-book-banner__subtitle {
    margin: 0 0 14px 0;
    font-size: 14px;
    line-height: 1.55;
    color: var(--muted);
    max-width: 62ch;
  }

  .azam-book-banner__actions {
    display: flex;
    gap: 12px;
    align-items: center;
    flex-wrap: wrap;
  }

  .azam-book-banner__button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 10px 14px;
    border-radius: 12px;
    font-weight: 700;
    font-size: 14px;
    color: #071018;
    text-decoration: none;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border: 0;
    box-shadow: 0 10px 22px rgba(0,0,0,0.28);
    transition: transform 140ms ease, filter 140ms ease;
  }

  .azam-book-banner__button:hover {
    transform: translateY(-1px);
    filter: brightness(1.02);
  }

  .azam-book-banner__link {
    font-size: 14px;
    color: rgba(255,255,255,0.86);
    text-decoration: none;
    border-bottom: 1px solid rgba(255,255,255,0.22);
    padding-bottom: 2px;
    transition: border-color 140ms ease, color 140ms ease;
  }

  .azam-book-banner__link:hover {
    color: rgba(255,255,255,0.95);
    border-color: rgba(255,255,255,0.45);
  }

  /* Mobile */
  @media (max-width: 520px) {
    .azam-book-banner__inner {
      grid-template-columns: 1fr;
      text-align: left;
    }

    .azam-book-banner__cover {
      justify-content: flex-start;
    }

    .azam-book-banner__cover img {
      width: 120px;
    }
  }
</style>