# Treasure Hunt Game (TrHuGa or TRaaS) on Rails.

## Idea

The idea is to have an application which hosts N Treasure Hunts, focussed on kids who might not be able to read correctly and might not speak English. An LLM will help by translating content.

The idea is that the parent creates a game, which is stored in the DB, and the child plays it. We'll call parent / kid the two roles, but this can of course be extended. what we mean really is parent=**organizer** and kid/child = **player**.

Design an app, in **Ruby on Rails** (Rails 8+, Ruby '3.4', TailwindCSS, `importmap` for JS management, Turbo and ActiveStorage), which hosts a number of games.

Models:

* **Game**. A game is a complicated object which contains .
    * A unique/mnemonic case-insensitive `public_code`, like in Slido. This is a "redirect" to the game path.
      It should be mnemonic, so a big friendly 6-letter code like "R7M5CH" to be entered in home page.
      This is generated once upon object creation and can't be modified by user.
    * A `start_date` and `end_date`. Game is invisible outside this, unless you're the owner or an admin.
    * A **published** (boolean, dflt=true). A game is playable only if its published, and StartDate <= TODAY <= EndDate.
    * an array of **clues** (see below).
    * Game Validation should fail unless the clues have all consecutive 1..N `series_id`. Example: [1,2,3] is good. [2,3,4] and [1,2,4,5] are bad.
    * **DefaultClueType**. This is just to help UX.
    * A **Context** (long TEXT) where the parents say everything about the event: who's birthday it is, how old are the kids, whats the occasion, and so on. This will be helpful for an LLM to talk to the kids.

* **User**. (via Devise). A user owns a game, and can edit it and see all about it.
    * a boolean is_admin (dflt=False). `db/seed.rb` will define a first user called "palladiusbonton@gmail" with some hardcoded hard password and admin-true. Every other user will not be an admin (meaning the signup flow wont allow it, of course).
    * A 2-letter language (eg 'en', 'it', 'fr', ..). Validate this corresponds to a valid language, possibly allowing to choose from N existing languages.
    * Game will be played mostly by unauthenticated users, so we need people to "log in" without an email.

* **Clue**. A clue has:
    * a contiguous ordered integer `series_id`. Clue 1 leads to Clue 2 which leads to Clue 3 and so on.
    * A 4-digit unique code `unique_code` like "7193". This needs to be unique across all clues with a single game (Small enough a 4y old can type it, but big enough not to play by exaustion).
    * A **parent_advisory**. Instructions for parents: where do you place the clue.
    * A **published** (boolean, dflt=true). I might play with some ideas but dont want to publish this yet.
    * A **ClueType**. Clue can be of two types: **QuestionAnswer** or **Physical**. Use an ENUM for this, so we can easily extend in the future.
        * A *QuestionAnswer* clue is done to be managed purely online (eg, from an ipad). The DB will store **question** (string), **answer** (string) and an LLM will validate it (ie, we don't want to be too picky if kid misspells the word "elephant").
            * an additional "visual_description" string (optional) will allow an LLM to paint the clue if needed.
        * A *Physical* clue is managed to be hidden from your kids somewhere, eg in the bedroom or in a SpielPlatz. Such a clue will have these info:
            * A **next_clue_riddle**. Instructions for kids: where to find the the clue.
            * A **location** string, think of a Google Maps address, like "Time Square, NYC".
            * A **geo_x** and **geo_y** floats, which identify the place as above. They should be semantically the same, one good for humans, one good for computers and to pinpoint on maps. I expect for user to type something, have autocompletion, and once it autocompletes it saves all 3 (location and geo_x/y).
            * A **location_addon** string, for kids, which says where exactly in "Times Square" you can find the piece of paper we'll hide for you. Eg, "by the traffic light in front of Starbucks".

## UX

Game experience. 99% the game is experienced with NO password using the initial code.

**Player UX**

* User will choose a language amongst English, French, Italian, Portuguese, German, Polish and Japanese.
* They will also choose a nickname.
* Ideally, the CUJ is that the kid is given a code from parents and they put it on the `/` app, and they start the game. The game should be able to speak with microphone and ask questions (using Gemini TTS capabilities).
* Once the game is open, the first clue is shown with the first question. Then the kid can answer the riddle (if QuestionAnswer), or look for the next token.
* For *Physical* games, we also need the ability to print in paper. The printout in PdF should have, say 4 clue per page and each clue should have:
    * a square containing both the kid's riddle for next step AND the CODE (so if they can read, they read it. If they can't, they just put the code on the app, see the next step, receive some sort of congratulation and get the next step read out loud for them).
* The app should be able to navigate back and forth the "seen" clues. Say we are at clue 3, we can navigate back and forth 1 <=> 2 <=> 3 but can't navigate to 4 until we give the answer. Page should always have some sort of textbox where we can put the answer (either a code, if physical, or the answer if Q&A).
* If Q&A, I expect some sort of chat inrteraction with the LLM which validates and decides if its right or not. If right, say the answer is elephant, and kid types "elefant", the LLM could be happy and give out the code for next step, say 0042.

For the parent/organizer:

* A Google Map with all the hints should be available, and a view of open games with that id (Turbo can help with that). Ideally, we'd have all clues as yellow pointers in a map which is big enough to contain them all, and a beeping blue pointer which moves and is now - say - at step 3.
* there will be a real time list of nicknames and completion percentage, eg "Riccardo: 3/18. Eleonora: 5/18".

## Files

* Create an `.env.dist` file with sample values, see below for what goes in it.
* DB will have to work on a vanilla Sqlite initially, but we already need the gems for postgreS since we'll have that in prod. We'll use a `DATABASE_URL` to specify it.
* Create a `justfile` for common commands. First command will be `list -> just -l` of course.

## Deployment

The script will work on Google Cloud, so we need a `Dockerfile`.

We use Google Cloud whenever possible:

* GCS for ActiveStorage.
* Cloud Run for deployments
* Cloud Build / Artifact Repository for build part.
* Gemini 2.0 / 2.5 flash as LLMs. `GEMINI_API_KEY` for simplicity.
* Cloud SQL + PostgreS for prod/db.

Please create a startup script and then give me instructions on how to proceed to change. I'd love to have a sample game created in db/seed.rb. One game based on Q&A and one based on physical clues around the city of Zurich, Switzerland.
