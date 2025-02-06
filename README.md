# Chunkr AI Tablebench


This is a bench on RD Table bench (https://huggingface.co/datasets/reducto/rd-tablebench) for the Chunkr AI via our new API. The results in the grading repo (https://github.com/reductoai/rd-tablebench), huggingface, and blog, are outdated by a few months. We are publishing our results here with our currrent API, we score 0.81 (on about half the dataset) compared to the <0.65 on the original benchmark run. 

We ran the dataset with the following configuration on our API: 
```
    config = Configuration(
        ocr_strategy=OcrStrategy.ALL,
        segmentation_strategy=SegmentationStrategy.PAGE,
        segment_processing=SegmentProcessing(page=GenerationConfig(html=GenerationStrategy.LLM)),
        high_resolution=True
    )
```

### Some notes/limitations we noticed on RD Tablebench and its implementation:

- String matching for table grading has significant limitations:
  - Fails to capture the visual and semantic structure of tables
  - Cannot evaluate layout and formatting that conveys meaning
  - Relies on brittle heuristics that break on valid variations

- The scoring criteria in grading.py (S_ROW_MATCH, G_ROW, etc.) (in ```grading.py``` in their github repo) may not reflect real-world table quality. For example, intelligently merged columns that maintain semantic meaning are heavily penalized, even though they may be perfectly valid for downstream LLM tasks.
The code only strips newlines, hyphens, and whitespace, but doesn't handle other HTML formatting like:
- ```<b>``` or ```<strong>``` tags for bold text
- ```<i>``` or ```<em>``` tags for italics
- ```<sup>``` and ```<sub>```for superscript/subscript
Such differences are not normalized but are not consistent across the dataset either - which skews results (see ```edge_case_001.png```).

- Character Normalization Issues:
  - No handling of special characters or their HTML entities (e.g., ```&amp;``` vs &)
  - No Unicode normalization (e.g., composed vs decomposed characters)

### Conclusion

The ground truth is also not always representative of the tables. Despite Gemini-Pro-1.5 (what we use for tables) scoring lower than Reducto, in practice it is actually better some of the ground truth samples. I've saved one such example to the ```edge_case_001.png``` file. The ground truth incorrectly centers the title and does not add bold tags to the first column's rows. We get penalized for doing this correctly. Another evaluator (who unfortunately has outdated chunkr results) also independently found discrepancies in the original benchmark (https://www.sergey.fyi/articles/gemini-flash-2). It is likely that a higher score on this benchmark (>85%) correlates with worse real-world performance/indication of overfitting on the eval. 
