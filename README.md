# Chunkr AI Tablebench


This is a bench on RD Table bench (https://huggingface.co/datasets/reducto/rd-tablebench) for the Chunkr AI via our new API. The results in this repo and blog dare outdated by a few months - so we are publishing our results here. On the new implementation we scored 0.81 compared to the <0.65 on the original benchmark run.We are publishing our results on the dataset here: 
    config = Configuration(
        ocr_strategy=OcrStrategy.ALL,
        segmentation_strategy=SegmentationStrategy.PAGE,
        segment_processing=SegmentProcessing(page=GenerationConfig(html=GenerationStrategy.LLM)),
        high_resolution=True
    )

### Some notes/limitations we noticed on RD Tablebench and its implementation:

- String matching for table grading has significant limitations:
  - Fails to capture visual and semantic structure of tables
  - Cannot evaluate layout and formatting that conveys meaning
  - Relies on brittle heuristics that break on valid variations

- The scoring criteria in grading.py (S_ROW_MATCH, G_ROW, etc.) (in ```grading.py``` in their github repo https://github.com/reductoai/rd-tablebench) may not reflect real-world table quality. For example, intelligently merged columns that maintain semantic meaning are heavily penalized, even though they may be perfectly valid for downstream LLM tasks.
The code only strips newlines, hyphens, and whitespace, but doesn't handle other HTML formatting like:
- ```<b>``` or ```<strong>``` tags for bold text
- ```<i>``` or ```<em>``` tags for italics
- ```<sup>``` and ```<sub>```for superscript/subscript
Minor differences are not normalized, but are not consistent accross the dataset either - which skews results (see ```/edge_case```)

- Character Normalization Issues:
  - No handling of special characters or their HTML entities (e.g., ```&amp;``` vs &)
  - No unicode normalization (e.g., composed vs decomposed characters)

### Conclusion

The ground truth is also not truly representative of the tables. Despite Gemini-Pro-1.5 (what we use for tables) scoring lower than Reducto, in practice it is actually better than the ground truth. I've added one such example to the ```/edge_cases``` folder - we get less than 0.5 for this sample, although it is actually better than the ground truth. The ground truth incorrectly centers the title, and does not add bold tags to the first column's rows. We get penalized for doing this correctly. It is likely that a higher score on this benchmark (>85%) actually correlates with worse real world performance/indication of over fitting on the training set. 
