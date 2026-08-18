# Notebook

This notebook contains a lot of information after hours and hours of testing different things about different frameworks and libraries. It is a collection of notes, code snippets, and references that I found useful during my development journey.

## Create or update a skill reference
Claude is a large language model that can load references with its skills feature.

your job is to create a skill reference that contains the information in this notebook.

You can only create or update a skill reference for the following artifacts:

- NestJS

If I ask you to create or update a skill reference for any other artifact, you should respond with "I can only create or update a skill reference for [list of artifacts]."

As you can see, each artifact is a separate folder in this repository. Each artifact must have an `ai` folder that contains a `skill-reference` subfolder.

Your job when I tell you to create or update a skill reference is to create or update the `skill-reference` subfolder for the specified artifact. The `skill-reference` subfolder should contain an optimized version of each file in the artifact's folder, with every single subfolder and file included. Dont include any files or folders that are not part of the artifact's folder or any index file (Example the index.md is the file in charge to navigate through the artifact's folder) I just need info about how to do things.
The original artifact's files contain a lot of links to other files and folders, but you should not include any links in the skill reference.

Each reference file should be located in the root of the `skill-reference` subfolder. Any subfolders in the artifact's folder should be flattened into the root of the `skill-reference` subfolder. The file names in the `skill-reference` subfolder should be descriptive for the information it contains.

Also, in skill references, you should include an index file that contains a list of all the files in the skill reference, with links to each file. In this way, if I want to make any changes to the skill reference, when updating the SKILL.md, AI can easily know which files have been added, removed, or modified.

IMPORTANT: It is not your business to create any SKILL.md files. You should only create or update the skill reference for the specified artifact, so the SKILL.md file can read the skill reference properly.

