# Radarr & Sonarr German Dual Language Quality Profiles & Custom Formats Guide

This guide outlines how to configure and fully automate Radarr and Sonarr to prefer German + English dual releases.

Last Updated: 2023-07-11

# [Deutsche Anleitung](https://github.com/PCJones/radarr-sonarr-german-dual-language/blob/main/GERMAN_DUAL_LANGUAGE_GUIDE_GER.md)

## Table of Contents
- [Contributing](#contributing)
- [Important Note](#important-note)
- [General Information](#general-information)
- [I don't want dual language, I want to prefer German but use English as fallback (or vice versa)](#i-dont-want-dual-language-i-want-to-prefer-german-but-use-english-as-fallback-or-vice-versa)
- [Guide](#guide)
  - [1. Import the German DL Custom Format](#1-import-the-german-dl-custom-format)
  - [2. Import the Language: Not ENG/GER Custom Format](#2-import-the-language-not-engger-custom-format)
  - [3. Create a Quality Profile](#3-create-a-quality-profile)
  - [4. Merge Quality](#4-merge-quality)
  - [5. Set the Language](#5-set-the-language)
  - [6. Set Upgrade Until Custom](#6-set-upgrade-until-custom)
  - [7. Set Custom Format Scores](#7-set-custom-format-scores)
  - [8. Prefer Language](#8-prefer-language)
    - [Prefer German over English if there is no DL Release](#--prefer-german-over-english-if-there-is-no-dl-release)
    - [Prefer English over German if there is no DL Release](#--prefer-english-over-german-if-there-is-no-dl-release)
  - [9. Quality Upgrades via Custom Formats](#9-quality-upgrades-via-custom-formats)
  - [10. Optional: Avoid Mic Dubbed Releases](10-optional-avoid-mic-dubbed-releases)
- [Contact & Support](#contact--support)

## Contributing
If you have questions, problems or improvements don't hesitate to create an issue or pull request!

## Important Note
This setup requires Sonarr v4 Beta. Sonarr v3 will not work. However, Sonarr v4 Beta is quite stable and should cause few, if any, problems.

## General Information
To reliably find German + English Dual Language releases (from now on referred to as "German DL" as the German Scene refers to them), it's best to have an indexer that specializes in these types of releases.

## I don't want dual language, I want to prefer German but use English as fallback (or vice versa)
No problem at all! Just follow every step in the guide but skip every step about the "German DL" custom format. At [Prefer Language](#8-prefer-language) import the one that fits your setup and then **remove the "NOT German DL" condition** before saving. Sorry if this sounds complicated, I'll soon create an extra page just for this scenario!

## Guide

### 1. Import the German DL Custom Format
Import the Custom Format "German DL":

```json
{
  "name": "German DL",
  "includeCustomFormatWhenRenaming": true,
  "specifications": [
    {
      "name": "German DL",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": "(?i)german\\s*\\.?dl|(?<=\\bGerman\\b.*)(?<!\\bWEB[-_. ])\\bDL\\b|\\[DE\\+[a-z]{2}\\]|\\[[a-z]{2}\\+DE\\]|ger,\\s*[a-z]{3}\\]|\\[[a-z]{3}\\s*,\\s*ger\\]"
      }
    }
  ]
}
```
This custom format matches all possible combinations of "German DL" (without being falsely triggered by WEB-DL). It also matches combinations of [ger,eng] and [DE+EN] which can be found on some torrents.

### 2. Import the Language: Not ENG/GER Custom Format
Import the Custom Format "Language: Not ENG/GER":

```json
{
  "name": "Language: Not ENG/GER",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Not English Language",
      "implementation": "LanguageSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": 1
      }
    },
    {
      "name": "Not German Language",
      "implementation": "LanguageSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": 4
      }
    },
    {
      "name": "Not German in Title",
      "implementation": "ReleaseTitleSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": "(?i)\\bgerman\\b"
      }
    }
  ]
}
```
This custom format is there to exclude any languages other than German and English.

### 3. Create a Quality Profile
Create a Quality Profile based on the Trash Guides for [Radarr](https://trash-guides.info/Radarr/radarr-setup-quality-profiles/#trash-quality-profiles) and [Sonarr](https://trash-guides.info/Sonarr/sonarr-setup-quality-profiles/). If you're already familiar with setting up quality profiles, you may not need to follow these guides exactly, but they're recommended for those new to the process.

### 4. Merge Quality

In order to prefer German DL over quality, we need to merge all desired qualities into a single group. 
![merge animation](https://trash-guides.info/Radarr/Tips/images/merge.gif)

How to merge qualities - *animation from [TRaSH Guides](https://trash-guides.info/Radarr/Tips/Merge-quality/) ([MIT License](https://github.com/TRaSH-Guides/Guides/blob/master/LICENSE))*.

In this example we'll want to prefer the quality in this descending order: Remux-1080p -> Bluray-1080p -> WEBDL-1080 -> Bluray-720p

Here is how it should look after merging the qualities:

![Merge Qualities 1](https://raw.githubusercontent.com/PCJones/radarr-sonarr-german-dual-language/main/img/merge_qualities_1.png)
![Merge Qualities 2](https://raw.githubusercontent.com/PCJones/radarr-sonarr-german-dual-language/main/img/merge_qualities_2.png)

Despite merging the qualities, it's still possible to upgrade them through custom formats. More details on that can be found in the section [Quality Upgrades via Custom Formats](#8-quality-upgrades-via-custom-formats).

### 5. Set the Language
The language in the quality profile profile must be set to `Any`.

### 6. Set Upgrade Until Custom
In your quality profile set the value of "Upgrade Until Custom" to `50000`

### 7. Set Custom Format Scores
In the Quality Profile settings, set the scores for the custom formats as follows:

| Custom Format         | Score  |
|-----------------------|--------|
| German DL             | 25000  |
| Language: Not ENG/GER | -30000 |

### 8. Prefer Language

#### - Prefer German over English if there is no DL Release
This will result in this priority:
1. German + English language
2. German language
3. English language

Import the Custom Format "Language: German Only":

```json
{
  "name": "Language: German Only",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Language GER",
      "implementation": "LanguageSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 4
      }
    },
    {
      "name": "NOT German DL",
      "implementation": "ReleaseTitleSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": "(?i)german\\s*\\.?dl|(?<=\\bGerman\\b.*)(?<!\\bWEB[-_. ])\\bDL\\b|\\[DE\\+[a-z]{2}\\]|\\[[a-z]{2}\\+DE\\]|ger,\\s*[a-z]{3}\\]|\\[[a-z]{3}\\s*,\\s*ger\\]"
      }
    }
  ]
}
```
Set the custom score to `15000`.


#### - Prefer English over German if there is no DL Release
This will result in this priority:
1. German + English language
2. English language
3. German language

Import the Custom Format "Language: English Only":

```json
{
  "name": "Language: English Only",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Language ENG",
      "implementation": "LanguageSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 1
      }
    },
    {
      "name": "NOT German DL",
      "implementation": "ReleaseTitleSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": "(?i)german\\s*\\.?dl|(?<=\\bGerman\\b.*)(?<!\\bWEB[-_. ])\\bDL\\b|\\[DE\\+[a-z]{2}\\]|\\[[a-z]{2}\\+DE\\]|ger,\\s*[a-z]{3}\\]|\\[[a-z]{3}\\s*,\\s*ger\\]"
      }
    }
  ]
}
```
Set the custom score to `15000`.

### 9. Quality Upgrades via Custom Formats

We want to prioritize German DL releases and upgrade to higher qualities within that subset.

Import the Custom Formats you need:

<details>
<summary><b>Bluray-1080p</b></summary>

```json
{
  "name": "Bluray-1080p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Bluray",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 9
      }
    },
    {
      "name": "1080p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 1080
      }
    },
    {
      "name": "Not REMUX",
      "implementation": "QualityModifierSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": 5
      }
    }
  ]
}
```
</details>

<details>
<summary><b>Bluray-2160p</b></summary>

```json
{
  "name": "Bluray-2160p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Bluray",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 9
      }
    },
    {
      "name": "2160p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 2160
      }
    },
    {
      "name": "Not REMUX",
      "implementation": "QualityModifierSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": 5
      }
    }
  ]
}
```
</details>

<details>
<summary><b>Bluray-720p</b></summary>

```json
{
  "name": "Bluray-720p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Bluray",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 9
      }
    },
    {
      "name": "720p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 720
      }
    },
    {
      "name": "Not REMUX",
      "implementation": "QualityModifierSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": 5
      }
    }
  ]
}
```
</details>

<details>
<summary><b>WEBDL-1080p</b></summary>

```json
{
  "name": "WEBDL-1080p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {


      "name": "WEBDL",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 7
      }
    },
    {
      "name": "1080p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 1080
      }
    }
  ]
}
```
</details>

<details>
<summary><b>WEBDL-2160p</b></summary>

```json
{
  "name": "WEBDL-2160p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "WEBDL",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 7
      }
    },
    {
      "name": "2160p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 2160
      }
    }
  ]
}
```
</details>

<details>
<summary><b>WEBDL-720p</b></summary>

```json
{
  "name": "WEBDL-720p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "WEBDL",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 7
      }
    },
    {
      "name": "720p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 720
      }
    }
  ]
}
```
</details>

<details>
<summary><b>WebRip-1080p</b></summary>

```json
{
  "name": "WebRip-1080p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "WebRip",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 4
      }
    },
    {
      "name": "1080p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 1080
      }
    }
  ]
}
```
</details>

<details>
<summary><b>WebRip-720p</b></summary>

```json
{
  "name": "WebRip-720p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "WebRip",
      "implementation": "SourceSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 4
      }
    },
    {
      "name": "720p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 720
      }
    }
  ]
}
```
</details>

<details>
<summary><b>Remux-1080p</b></summary>

```json
{
  "name": "Remux-1080p",
  "includeCustomFormatWhenRenaming":

 false,
  "specifications": [
    {
      "name": "1080p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 1080
      }
    },
    {
      "name": "REMUX",
      "implementation": "QualityModifierSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 5
      }
    }
  ]
}
```
</details>

<details>
<summary><b>Remux-2160p</b></summary>

```json
{
  "name": "Remux-2160p",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "2160p",
      "implementation": "ResolutionSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 2160
      }
    },
    {
      "name": "REMUX",
      "implementation": "QualityModifierSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": 5
      }
    }
  ]
}
```
</details>

> Note: If you require a custom format for a different quality, import one of the above examples and modify the conditions accordingly. It should be relatively straightforward.

In the [previous example](https://raw.githubusercontent.com/PCJones/radarr-sonarr-german-dual-language/main/img/merge_qualities_2.png) where we looked for Remux-1080p, Bluray-1080p, WEBDL-1080p, and Bluray-720p, we would need to add a custom format for each of these.

Next, assign scores to these custom formats in the quality profile. Assign `0` for the lowest quality, `2000` for the second lowest, `4000` for the next, and so on, increasing the score by `2000` each time. This scoring system will prioritize and upgrade to higher quality formats when they become available. 

Here is an example of what the scoring should look like:

![Custom Format Quality](https://github.com/PCJones/radarr-sonarr-german-dual-language/blob/main/img/custom_format_quality.png?raw=true)

Now, whenever a higher-quality German DL release becomes available, your setup will automatically upgrade to it.

### 10. Optional: Avoid Mic Dubbed Releases
Mic dubbed releases are poor quality sound recordings made in movie theaters, for example, and may include sounds made by people there. Very often the first German DL release will have great english soundtrack but a mic dubbed German soundtrack. Unless you are really impatient I highly recommend to avoid them. This is only needed for Radarr, not for Sonarr.

Import the MIC DUB Custom Format:

<details>
<summary><b>MIC DUB Custom Format</b></summary>

```json
{
  "name": "MIC DUB",
  "includeCustomFormatWhenRenaming": true,
  "specifications": [
    {
      "name": "MD",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": "((19|20|21)[0-9]{2}).*?(?<=^|[ .-])MD(?=[ .-]|$).*?(?<=^|[ .-])GERMAN(?=[ .-]|$)|((19|20|21)[0-9]{2}).*?(?<=^|[ .-])GERMAN(?=[ .-]|$).*?(?<=^|[ .-])MD(?=[ .-]|$)"
      }
    }
  ]
}
```
</details>

In the Quality Profile settings, set the scores for the custom format as follows:

| Custom Format         | Score  |
|-----------------------|--------|
| MIC DUB             | -25000  |

## Contact & Support
- Feel free to create an Issue if you need support. 
- [Telegram](https://t.me/pc_jones)
- Discord: pcjones1
