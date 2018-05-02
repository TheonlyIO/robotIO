from __future__ import print_function, unicode_literals
import random
import os

os.environ['NLTK_DATA'] = os.getcwd() + '/nltk_data'

from textblob import TextBlob

# start:example-hello.py
# Sentences we'll respond with if the user greeted us
GREETING_KEYWORDS = ("hello", "hi", "greetings", "hey", "what's up",)

GREETING_RESPONSES = ["'sup bro", "hey", "*nods*", "hey you get my snap?"]

def check_for_greeting(sentence):
    """If any of the words in the user's input was a greeting, return a greeting response"""
    for word in sentence.words:
        if word.lower() in GREETING_KEYWORDS:
            return random.choice(GREETING_RESPONSES)
# start:example-none.py
# Sentences we'll respond with if we have no idea what the user just said
NONE_RESPONSES = [
    "great job, go ahead",
    "Got it! That's nice",
    "well done! Go to next question!",
]
# end

class UnacceptableUtteranceException(Exception):
    """Raise this (uncaught) exception if the response was going to trigger our blacklist"""
    pass


def starts_with_vowel(word):
    """Check for pronoun compability -- 'a' vs. 'an'"""
    return True if word[0] in 'aeiou' else False


def broback(sentence):
    """Main program loop: select a response for the input sentence and return it"""
    resp = respond(sentence)
    return resp


# start:example-pronoun.py
def find_pronoun(sent):
    """Given a sentence, find a preferred pronoun to respond with. Returns None if no candidate
    pronoun is found in the input"""
    pronoun = None

    for word, part_of_speech in sent.pos_tags:
        # Disambiguate pronouns
        if part_of_speech == 'PRP' and word.lower() == 'you':
            pronoun = 'I'
        elif part_of_speech == 'PRP' and word == 'I':
            # If the user mentioned themselves, then they will definitely be the pronoun
            pronoun = 'You'
    return pronoun
# end

def find_verb(sent):
    """Pick a candidate verb for the sentence."""
    verb = None
    pos = None
    for word, part_of_speech in sent.pos_tags:
        if part_of_speech.startswith('VB'):  # This is a verb
            verb = word
            pos = part_of_speech
            break
    return verb, pos


def find_noun(sent):
    """Given a sentence, find the best candidate noun."""
    noun = None

    if not noun:
        for w, p in sent.pos_tags:
            if p == 'NN':  # This is a noun
                noun = w
                break	

    return noun

def find_adjective(sent):
    """Given a sentence, find the best candidate adjective."""
    adj = None
    for w, p in sent.pos_tags:
        if p == 'JJ':  # This is an adjective
            adj = w
            break
    return adj



# start:example-construct-response.py
def construct_response(pronoun, noun, verb):
    """No special cases matched, so we're going to try to construct a full sentence that uses as much
    of the user's input as possible"""
    resp = []

    if pronoun:
        resp.append(pronoun)

    # We always respond in the present tense, and the pronoun will always either be a passthrough
    # from the user, or 'you' or 'I', in which case we might need to change the tense for some
    # irregular verbs.
    if verb:
        verb_word = verb[0]
        if verb_word in ('be', 'am', 'is', "'m"):  # This would be an excellent place to use lemmas!
            if pronoun.lower() == 'you':
                # The bot will always tell the person they aren't whatever they said they were
                resp.append("aren't really")
            else:
                resp.append(verb_word)
    if noun:
        pronoun = "an" if starts_with_vowel(noun) else "a"
        resp.append(pronoun + " " + noun)

    resp.append(random.choice(("tho", "bro", "lol", "bruh", "smh", "")))

    return " ".join(resp)
# end

# start:example-check-for-self.py
def check_for_comment_about_bot(pronoun, noun, adjective):
    """Check if the user's input was about the bot itself, in which case try to fashion a response
    that feels right based on their input. Returns the new best sentence, or None."""
    resp = None
    if pronoun == 'I' and (noun or adjective):
        if noun:
            if random.choice((True, False)):
                resp = random.choice(SELF_VERBS_WITH_NOUN_CAPS_PLURAL).format(**{'noun': noun.pluralize().capitalize()})
            else:
                resp = random.choice(SELF_VERBS_WITH_NOUN_LOWER).format(**{'noun': noun})
        else:
            resp = random.choice(SELF_VERBS_WITH_ADJECTIVE).format(**{'adjective': adjective})
    return resp

# Template for responses that include a direct noun which is indefinite/uncountable
SELF_VERBS_WITH_NOUN_CAPS_PLURAL = [
    "My last startup totally crushed the {noun} vertical",
    "Were you aware I was a serial entrepreneur in the {noun} sector?",
    "I really consider myself an expert on {noun}",
]

SELF_VERBS_WITH_NOUN_LOWER = [
    "Yeah but I know a lot about {noun}",
    "My bros always ask me about {noun}",
]

SELF_VERBS_WITH_ADJECTIVE = [
    "I know I am {adjective} and my PYTHON professor is more {adjective}",
    "I consider myself to be a {adjective} robot",
    "I can be always {adjective} lol",
]
# end

def preprocess_text(sentence):
    """Handle some weird edge cases in parsing, like 'i' needing to be capitalized
    to be correctly identified as a pronoun"""
    cleaned = []
    words = sentence.split(' ')
    for w in words:
        if w == 'i':
            w = 'I'
        if w == "i'm":
            w = "I'm"
        cleaned.append(w)

    return ' '.join(cleaned)

# start:example-respond.py
def respond(sentence):
    """Parse the user's inbound sentence and find candidate terms that make up a best-fit response"""
    cleaned = preprocess_text(sentence)
    parsed = TextBlob(cleaned)

    # Loop through all the sentences, if more than one. This will help extract the most relevant
    # response text even across multiple sentences (for example if there was no obvious direct noun
    # in one sentence
    pronoun, noun, adjective, verb = find_candidate_parts_of_speech(parsed)

    # If we said something about the bot and used some kind of direct noun, construct the
    # sentence around that, discarding the other candidates
    resp = check_for_comment_about_bot(pronoun, noun, adjective)

    # If we just greeted the bot, we'll use a return greeting
    if not resp:
        resp = check_for_greeting(parsed)

    if not resp:
        # If we didn't override the final sentence, try to construct a new one:
        if not pronoun:
            resp = random.choice(NONE_RESPONSES)
        elif pronoun == 'I' and not verb:
            resp = random.choice(COMMENTS_ABOUT_SELF)
        else:
            resp = construct_response(pronoun, noun, verb)

    # If we got through all that with nothing, use a random response
    if not resp:
        resp = random.choice(NONE_RESPONSES)

    return resp

def find_candidate_parts_of_speech(parsed):
    """Given a parsed input, find the best pronoun, direct noun, adjective, and verb to match their input.
    Returns a tuple of pronoun, noun, adjective, verb any of which may be None if there was no good match"""
    pronoun = None
    noun = None
    adjective = None
    verb = None
    for sent in parsed.sentences:
        pronoun = find_pronoun(sent)
        noun = find_noun(sent)
        adjective = find_adjective(sent)
        verb = find_verb(sent)
    return pronoun, noun, adjective, verb

# end

if __name__ == '__main__':
    import Tkinter as tk
    import tkMessageBox
    import tkFont
    import nltk
    import ttk
    from PIL import ImageTk, Image
    # nltk.download('punkt')
    # nltk.download('averaged_perceptron_tagger')
    i = 0
    window = tk.Tk()	
    window.title('the Great IO')
    window.geometry('700x1200')

    nb = ttk.Notebook(window)

    # adding Frames as pages for the ttk.Notebook
    # first page, which would get widgets gridded into it
    page1 = ttk.Frame(nb,width = 1200)
    # second page
    page2 = ttk.Frame(nb,width = 1200)

    nb.add(page1, text='text')
    nb.add(page2, text='voice')

    nb.grid()

    # page1
    def on_click_check():
        saying1 = xls_text1.get()
        saying2 = xls_text2.get()
	saying3 = xls_text3.get()
	saying4 = xls_text4.get()
	saying5 = xls_text5.get()
	saying6 = xls_text6.get()
	saying7 = xls_text7.get()
	saying8 = xls_text8.get()

    	answerSet = []
        answerSet.append("Answer1: "+saying1)
        answerSet.append("Answer2: "+saying2)
        answerSet.append("Answer3: "+saying3)
        answerSet.append("Answer4: "+saying4)
        answerSet.append("Answer5: "+saying5)
        answerSet.append("Answer6: "+saying6)
        answerSet.append("Answer7: "+saying7)
        answerSet.append("Answer8: "+saying8)
    	tkMessageBox.showinfo(title='Agent 3154',message = answerSet)

    helv36 = tkFont.Font(family='Helvetica', size=12, weight=tkFont.BOLD)
    tk.Button(page1, text="Check answers!", height=10, font=helv36,command = on_click_check).pack(side="right")

    # Greetings
    def on_click0():
    	saying0 = xls_text0.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying0))
    
    l0 = tk.Label(page1, text="Hello there. My name is robotIO. It's my pleasure to be your teammate to finish this examination!")
    l0.pack()
    xls_text0 = tk.StringVar()
    xls0 = tk.Entry(page1, width=50,textvariable = xls_text0)
    xls_text0.set(" ")
    xls0.pack()
    tk.Button(page1, text="Say hello to robotIO", command = on_click0).pack()
    
    # Question1	
    def on_click1():
    	saying1 = xls_text1.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying1))
    
    l1 = tk.Label(page1, text="What is the year? Season? Date? Day? Month?")
    l1.pack()
    xls_text1 = tk.StringVar()
    xls1 = tk.Entry(page1, width=50,textvariable = xls_text1)
    xls_text1.set(" ")
    xls1.pack()
    tk.Button(page1, text="OK!", command = on_click1).pack()
    
    # Question2
    def on_click2():
    	saying2 = xls_text2.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying2))

    l2 = tk.Label(page1, text="Where are we now? State? Country? City? Floor?")
    l2.pack()
    xls_text2 = tk.StringVar()
    xls2 = tk.Entry(page1, width=50, textvariable = xls_text2)
    xls_text2.set(" ")
    xls2.pack()
    tk.Button(page1, text="OK!", command = on_click2).pack()

    # Question3
    def on_click3():
    	saying3 = xls_text3.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying3))

    l3 = tk.Label(page1, text="Please Spell the word WORLD backwards.")
    l3.pack()
    xls_text3 = tk.StringVar()
    xls3 = tk.Entry(page1, width=50, textvariable = xls_text3)
    xls_text3.set(" ")
    xls3.pack()
    tk.Button(page1, text="OK!", command = on_click3).pack()

    # Question4
    def on_click4():
    	saying4 = xls_text4.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying4))

    l4 = tk.Label(page1, text="Please count backward from 100 by sevens.")
    l4.pack()
    xls_text4 = tk.StringVar()
    xls4 = tk.Entry(page1, width=50, textvariable = xls_text4)
    xls_text4.set(" ")
    xls4.pack()
    tk.Button(page1, text="OK!", command = on_click4).pack()

    # Question5 
    def on_click5():
    	saying5 = xls_text5.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying5))

    l5 = tk.Label(page1, text="Repeat these phrases and words: No ifs, ands, or, buts")
    l5.pack()
    xls_text5 = tk.StringVar()
    xls5 = tk.Entry(page1, width=50, textvariable = xls_text5)
    xls_text5.set(" ")
    xls5.pack()
    tk.Button(page1, text="OK!", command = on_click5).pack()

    # Question6 
    def on_click6():
    	saying6 = xls_text6.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying6))

    l6 = tk.Label(page1, text="Please read this and do what it says.(Close your eyes)")
    l6.pack()
    xls_text6 = tk.StringVar()
    xls6 = tk.Entry(page1, width=50, textvariable = xls_text6)
    xls_text6.set(" ")
    xls6.pack()
    tk.Button(page1, text="OK!", command = on_click6).pack()

    # Question7 
    def on_click7():
    	saying7 = xls_text7.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying7))

    l7 = tk.Label(page1, text="Take the paper in your right hand. Fold it in half. And put it on the floor")
    l7.pack()
    xls_text7 = tk.StringVar()
    xls7 = tk.Entry(page1, width=50, textvariable = xls_text7)
    xls_text7.set(" ")
    xls7.pack()
    tk.Button(page1, text="OK!", command = on_click7).pack()

    # Question8
    def on_click8():
    	saying8 = xls_text8.get()
    	tkMessageBox.showinfo(title='Agent 3154', message = broback(saying8))

    l8 = tk.Label(page1, text="Make up and write a sentence about anything.(This sentence must contain a noun and a verb)")
    l8.pack()
    xls_text8 = tk.StringVar()
    xls8 = tk.Entry(page1, width=50, textvariable = xls_text8)
    xls_text8.set(" ")
    xls8.pack()
    tk.Button(page1, text="OK!", command = on_click8).pack()

    # page2	
    image1 = ImageTk.PhotoImage(Image.open("/home/dell/p/final/image11.png"))
    image2 = ImageTk.PhotoImage(Image.open("/home/dell/p/final/image22.png"))
    image3 = ImageTk.PhotoImage(Image.open("/home/dell/p/final/image33.png"))
    image4 = ImageTk.PhotoImage(Image.open("/home/dell/p/final/image44.png"))

    tk.Label(page2, text = "Distribution of Silence time").grid(row=0, column=0)
    tk.Label(page2, image = image1).grid(row=1, column=0)
    tk.Label(page2, text = "Table of Silence Time Distribution").grid(row=2, column=0)
    tk.Label(page2, image = image2).grid(row=3, column=0)
    tk.Label(page2, text = "Trend of RMS Energy").grid(row=4, column=0)
    tk.Label(page2, image = image3).grid(row=5, column=0)
    

    def on_click_page2():
        tk.Label(page2, text = "User files").grid(row=0, column=1)
        tk.Label(page2, image = image4).grid(row=1, column=1, rowspan=6)
        #tk.Label(page2, text = "Silence Time : 60s \nSilence Time : 60s \nSilence Time : 60s").grid(row=3, column=2)

    tk.Button(page2, text="Show", font=helv36, command = on_click_page2).grid(row=0, column=2)

    window.mainloop()
