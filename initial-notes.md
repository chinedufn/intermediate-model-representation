# Intermediate Model Format

`.imf.json`

# Initial Motivation / Background

I want to least opinionated file format possible that still has all of the raw data that I need. I want it to be easy to write tooling around this format. I want linting and tests to be a major focus both for exporting to IMF and converting from IMF to other formats.

---

For the most part, a 3d model is just a set of numbers. The complexity comes from organizing these numbers so that they're useful for the end user. 

What if you could start with just the raw numbers and then gradually add opinions and static analysis yourself using off the shelf small modules?

Let's try and find out..

# Goals

- Give you all of the data about your model that you might need in a format that's dead simple for beginners to start using. 
    - You can do whatever you want with it.
    - Advanced users can go from this dumb format to something much more optimized for their needs by running it through 3rd party userland transforms.
        - Userland has tools to statically analyze your `IMF` model.
            - Random example... What is the volume of my model? How tall is it? How wide? How many animations? Average number of keyframes per animation? 
        - You might pick different minifiers off of the shelf depending on your use case
- No opinions about your data. Nothing is compressed. We just give you everything you need raw. Optimizing this file down is left up to userland.
- Imagine exporting a model from your 3d modeling software, running it through some off the shelf transforms and then having something that is perfect your use case.
- Should be super straightforward to go from `some 3d format -> IMF -> another 3d format` and vice versa. 
    - Every file format just needs to be able to convert to and from IMF
- Eliminate the problem of people exporting their models and struggling for hours and days to figure out why it isn't working. Why they can't open it in their engine / simulation.
    - EXTREMELY, MERCILLESLY strict linting on what is a valid IMF file. Tooling is key here. You should NEVER end up trying to use a model that will not give you what you're expecting.
- Make it easy to hook into the `IMF` exporter for different modeling programs.
    - People shouldn't need to write exporters for every modeling program when they create their own file format. Instead make it easy to wrap the IMF exporters so that they can transform the IMF data into their file format
    - IMF exporter should have unit tests like CRAZY. Needs to be reliable. We never want someone to export something and have it not be what they expected.
- IMF data should be organized so that it's easy to stream. This will help allow userland tools to be fast even for large models.
- IMF keeps it dead simple. Let 3rd party tools layer on any complexity and opinions that the user needs.
    - example. Only one model per valid IMF file. But you can easily append them yourself.
- Start off handling by use cases. If other people want to give it a try start accomodating their use cases in the most minimal way possible. Rinse and repeat.
    - We'll do a little of things wrong. But an emphasis on barely doing anything should make these mistakes easy to undo.

# Some Constraints

- Everything world entity in its own file. Every light, every armature, every mesh, every camera, etc
    - IMF's can reference eachother, but each is separate
    - Different models can share an armature by referencing it.
        - For example, if you have change-able clothing in your game, a t-shirt model, sweater model and jeans model can all share the same human skeletal armature by pointing to it
    - You may not want to serve a jillion files to your users... But that's the point! You can combine them in whatever way makes sense for your application! Over time, if we're lucky, you'll have off the shelf small tools written by others that help you do this.

# How to start

1. Jot down example file structures
2. Start writing specification that encompass these files
3. collada -> IMF converter
4. Figure it out from there

# Open Questions

## Handling IKs?

- Would need to export IK data and the name of the IK solver that is meant to be used since every modeling program uses its own solver. So we'd want it to be easy for a consumer to say "oh this model uses the "SuperIKSolver", ok I'll use that to solve the IKs for this model.
    - Not all 3d modeling programs are open source so this might not be possible for closed source modeling programs. Unless someone is able to reverse engineer their solvers. But that sounds.. hard?
    - For open source modeling programs like Blender, you can dig into the source and turn the solver into a module for different programming languages
        - @see chromakode's `fcurve` module as a reference example for this idea
- I need to do more research here. Haven't used IKs enough - so start by only supporting FK