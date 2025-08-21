# contentanalyser
N8N workflow that scrapes and uses AI to analyze content

Technical Documentation: Instagram Content Aggregator (Phase 1)
This document provides a technical overview of the automated workflows designed to aggregate and analyze content from specified Instagram profiles. The system is built using n8n.io for orchestration, Apify for data scraping, AI models for content analysis (OpenAI, TwelveLabs), and Airtable for data storage.

1. Workflow: "Instagram Reels Scraper and Analyzer"
Workflow Name: [Instagram Reels Scraper and Analyzer]
Objective:
This workflow automates the process of scraping Reels from specified Instagram profiles, conducting in-depth AI analysis, and storing the structured data in Airtable.

Execution Trigger:
The workflow is initiated by an Apify Trigger node, which is activated via a webhook after the corresponding Apify actor (Task: tihomirovevgen8888/instagram-story-details-scraper-task) completes its scraping run. The Apify actor is configured to run on a schedule.

Workflow Breakdown:
Apify Trigger & Get Dataset Items:
Function: Receives the webhook from Apify upon run completion and fetches the raw dataset of scraped Reels.
Importance: This is the entry point for all collected data into the n8n environment.

Normalize Apify Data (Code Node):
Function: A custom JavaScript node that processes the raw JSON output from Apify, ensuring data is correctly formatted and structured for seamless processing within n8n.
Filter (Conditional Node):
Function: Acts as a quality gate, validating each Reel against essential criteria: videoUrl must exist, errorDescription must not be "Post does not exist", and a unique id must be present.
Importance: Ensures that only valid and complete data proceeds to the resource-intensive AI analysis stage.

Loop Over Items (Batch Processing):
Function: Splits the incoming stream of Reels into small batches (e.g., 5 items per batch) and processes them sequentially with a delay between each batch.
Importance: This is a strategic implementation to mitigate rate limits and throttling from Instagram and third-party APIs (like TwelveLabs), preventing IP blocks and timeout errors during mass data processing.

Binary Video Download (HTTP Request Node):
Function: Downloads the video file for each Reel using its direct videoUrl.
Importance: This node is configured with specific headers, including Cookies and a User-Agent, to simulate legitimate user behavior and bypass Instagram's anti-scraping protections, ensuring stable video downloads.

TwelveLabs & OpenAI Integration (AI Analysis Pipeline):
twelve_labs (HTTP Request Node): Uploads the downloaded video file to the TwelveLabs API for video content analysis (transcription, indexing).
generate summary (HTTP Request Node): After a programmed delay (Wait node) to allow for processing, this node queries the TwelveLabs API to generate an AI-powered summary, identify topics, and extract highlights from the video.

AI - Summary, AI - Entity Extraction, etc. (OpenAI Nodes): This chain of nodes uses OpenAI's GPT-4o model to perform advanced text analysis on the video's caption and transcription, extracting a final summary, key entities, and sentiment.
Importance: This is the core "intelligence" of the system, transforming raw content into structured, actionable insights.

Create a Record (Airtable Node):
Function: Takes all the collected, processed, and enriched data for each Reel and creates a new, structured record in the designated Airtable base.
Importance: Serves as the final storage destination, creating an organized and accessible database for the client.

3. Workflow: "Instagram Stories Scraper and Analyzer"
Workflow Name: [Instagram Stories Scraper and Analyzer]
Objective & Logic:
This workflow operates on the same principles as the Reels scraper but is specifically configured for Instagram Stories.
It is triggered by a separate Apify actor designed for Stories.
The data processing pipeline is identical, with minor adjustments for the different data structure of Stories (e.g., handling Story Type instead of Likes/Comments).
The final enriched data is stored in a dedicated "Stories" table in Airtable.

5. Workflow: "Airtable Deduplicator"
Workflow Name: [Filter]
Objective:
This maintenance workflow ensures data integrity by automatically identifying and removing duplicate records from both the "Reels" and "Stories" tables in Airtable.

Execution Trigger:
The workflow is initiated by a Schedule Trigger node, configured to run periodically (e.g., every 5 days).

Workflow Breakdown:
Search Reels / Search Stories (Airtable Nodes):
Function: These nodes fetch all existing records from their respective tables in Airtable.
Filter Reels / Filter Stories (Code Nodes):
Function: A custom JavaScript node that processes all records to find duplicates.
Logic:
It groups records by a unique identifier (Post ID for Reels, Stories ID for Stories).
Within each group, it sorts the records by their creation time (createdTime field in Airtable).
It identifies all records except the oldest one as duplicates.

Delete Reels / Delete Stories (Airtable Nodes):
Function: Receives the list of Airtable Record IDs for all identified duplicates and permanently deletes them from the database.
Importance: This ensures the database remains clean and contains only unique content, which is crucial for accurate analysis.

Link to watch demo: https://youtu.be/mbYeRZm8rzE
