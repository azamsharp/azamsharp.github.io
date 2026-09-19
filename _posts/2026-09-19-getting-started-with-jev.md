# Getting Started with Jev: Choice, Score, and Noul Explained

Jev by TypeSafe is getting a lot of attention right now. Social media, especially Twitter, is filled with demos of people using Jev in many different and interesting ways. But if you are coming from traditional LLMs like ChatGPT, Claude, or Gemini, it may not be immediately clear what Jev does or when you should use it.

In this article, we will look at what Jev is and how it can be used for decision-making and structured output. We will also explore the three types of questions supported by Jev. These include **Choice, Score, and Noul**, which we will explore using practical, real-world examples.

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

### What is Jev?

Let's start with what Jev is not. Jev is not an LLM like ChatGPT, Claude, or Gemini. It does not produce text output. In other words, if you send your request to Jev asking, "List all 50 states in America," it will not return you a list of 50 states. That is not the point of Jev. For those kinds of questions, you can use your LLM of choice.

Jev is an AI model that is good at decision-making and providing structured output. Jev supports three different types of questions. These include:

1. Choice - Choose an option from a list.
2. Score - Score the state based on a provided rubric.
3. Noul - Is this statement true?

I think it is better to explain this with examples of each question type.

#### Choice

Consider a scenario where your company provides support through a chatbot on its website. Users can ask questions and get results. Behind the scenes, your company has implemented different AI profiles/assistants for different types of questions. One AI assistant is good at answering technical questions, another is good at answering billing questions, and the last one is good at answering general questions.

If the customer asks, "I returned the item last week. Why is my refund not processed?" which assistant do you think would be good at answering this specific question? If you answered **billing**, then you are correct.

That is the purpose of the Jev model: to make this decision and route the request/prompt to the correct assistant/profile. Below, you can see one of the requests to the Jev model.

```json
{
  "state": "I returned the item last week, why is my refund not processed?",
  "questions": {
    "category": {
      "type": "choice",
      "instructions": "Which support category does this request belong to?",
      "criteria": {
        "technical": "Technical problems with the application",
        "billing": "Payments, charges, refunds, or billing questions",
        "general": "General questions that do not belong to another category"
      }
    }
  }
}
```

The Choice question type, allows Jev model to select a single option. The selected option is based on the processed text. 

> Although you can use LLM to perform intent classificaton, but the results are hit or miss. Even if you use Foundation Models framework, which provides structured output most of the time the results were unexpected and took much longer to process.  

#### Score

Consider a scenario where, apart from routing the request to the correct assistant/profile, you are also looking for a measure of the customer's frustration. This can be accomplished by using **Score** questions.

If the user sends a message, "Where the hell is my order? I paid two weeks ago and it has still not arrived yet," we can set different criteria for our score. These can be:

* Calm
* Frustrated
* Very angry

Each criterion is associated with a number. So, Calm is 0, Frustrated is 1, and Very Angry is 2. If the score for the user's request comes out to be 0.1, then we can confidently say that the user is calm. On the other hand, if the score is 1.99, then it is close to 2, which means the user is very angry and we should take immediate action.

#### Noul

Consider a scenario where you are evaluating resumes for a senior SwiftUI developer position. You are looking for candidates who have SwiftUI, SwiftData, and networking experience. For these cases, you can use Noul questions.

Noul questions produce a numeric value. If the value is greater than or equal to 0.5, we can confidently say yes; otherwise, the answer is no. This means that for questions like, "Does this candidate have SwiftUI experience?" Jev can answer yes or no based on the resume. This can help you filter out candidates really quickly.

> You can watch the demo of the Resume Evaluator web app here: [Resume Evaluator Demo](https://x.com/azamsharp/status/2101325647142400224?s=20&utm_source=chatgpt.com)

### Resources 

- [TypeSafe AI Official Website](https://typesafe.ai/)
- [Jev Documentation](https://docs.typesafe.ai/introduction)
- [Getting Started with Jev: API Keys, HTTP Requests, JavaScript & Python SDKs](https://youtu.be/HaR_DdWrlhA?si=K7Xb0X2t7KTDNHi0)

### Conclusion

Jev provides a different way of working with AI. Instead of asking the model to generate text, we can use it to make decisions and return structured output based on the criteria we provide.

In this article, we looked at the three types of questions supported by Jev. Choice can be used to select an option from a list, Score can be used to evaluate something based on a rubric, and Noul can be used to determine whether a statement is true or false.

These simple question types can be used in many different real-world scenarios, including routing requests, measuring customer frustration, evaluating resumes, and much more.

### Continue Learning with AzamSharp School

Become an AzamSharp School member and get access to more than 250 hours of practical courses covering SwiftUI, SwiftData, testing, architecture, AI, machine learning, and more.

Your membership also includes access to AzamSharp books, live workshops, office hours, and new content added regularly.

[Join AzamSharp School](https://azamsharp.school?utm_source=chatgpt.com)
