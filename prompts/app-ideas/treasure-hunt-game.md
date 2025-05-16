# Treasure Hunt Game (TrHuGa or TRaaS)

The idea is to have an application which hosts N Treasure Hunts, focussed on kids who might not be able to read correctly and might not speak English. An LLM will help by translating content.

Design an app, in **Ruby on Rails** (Rails 8+, Ruby '3.4', TailwindCSS, `importmap` for JS management, Turbo and ActiveStorage), which hosts a number of games.

Models:

* **Game**. A game is a complicated object which contains .
    * A unique/mnemonic case-insensitive code, like in Slido. This is a "redirect" to the game path.
    * A start/end date. Game is invisible outside this delta_t, unless you're the owner or an admin.
    * an array of clues.
    * Game Validation should fail unless the clues have all consecutive 1..N `series_id`. Example: [1,2,3] is good. [2,3,4] and [1,2,4,5] are bad.
    * **DefaultClueType**. This is just to help UX.
* **User**. (via Devise). A user owns a game, and can edit it and see all about it.
    * a boolean is_admin (dflt=False). `db/seed.rb` will define a first user called "palladiusbonton@gmail" with some hardcoded hard password and admin-true. Every other user will not be an admin (meaning the signup flow wont allow it, of course).
    * A 2-letter language (eg 'en', 'it', 'fr', ..). Validate this corresponds to a valid language, possibly allowing to choose from N existing languages.
* **Clue**. A clue has:
    * a contiguous ordered integer `series_id`. Clue 1 leads to Clue 2 which leads to Clue 3 and so on.
    * A **ClueType**. Clue can be of two types: **QuestionAnswer** or **Physical**. Use an ENUM for this, so we can easily extend in the future.
        * A *QuestionAnswer* clue is done to be managed purely online (eg, from an ipad). The DB will store **question** (string), **answer** (string) and an LLM will validate it (ie, we don't want to be too picky if kid misspells the word "elephant")
        * A *Physical* clue is managed to be hidden from your kids somewhere, eg in the bedroom or in a SpielPlatz.

Game experience. 99% the game is experienced with NO password using.
* User will choose a language amongdst English, French, Italian, Portugese, German, Polish and Japanese.

* Create an `.env.dist` file with sample values, see below for what goes in it.

## Deployment

The script will work on Google Cloud, so we need a `Dockerfile`.

We use Google Cloud whenever possible:

* GCS for ActiveStorage.
* Cloud Run for deployments
* Cloud Build / Artifact Repository for build part.
* Gemini 2.0 / 2.5 flash as LLMs. `GEMINI_API_KEY` for simplicity.
