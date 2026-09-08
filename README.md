# MITRA
MITRA — A robust multi-modal image correspondence and registration system for Chandrayaan-2 optical imagery, designed for scale, illumination, and viewpoint variations

The system combines feature-based correspondence, geometric verification, spatially distributed matching, image registration, and quantitative evaluation into a single transparent workflow.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Objectives](#objectives)
- [MITRA Solution](#mitra-solution)
- [Core Pipeline](#core-pipeline)
- [Key Features](#key-features)
- [Why Spatial Distribution Matters](#why-spatial-distribution-matters)
- [Supported Imagery](#supported-imagery)
- [System Architecture](#system-architecture)
- [Registration Workflow](#registration-workflow)
- [Evaluation Metrics](#evaluation-metrics)
- [Baseline and MITRA Comparison](#baseline-and-mitra-comparison)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Frontend](#frontend)
- [Computer Vision Pipeline](#computer-vision-pipeline)
- [Data](#data)
- [Current Development Status](#current-development-status)
- [Roadmap](#roadmap)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [Installation](#installation)
- [Usage](#usage)
- [Research and Evaluation](#research-and-evaluation)
- [Smart India Hackathon 2026](#smart-india-hackathon-2026)
- [Team](#team)
- [Disclaimer](#disclaimer)
- [License](#license)

---

# Overview

Planetary image registration is an important step in combining, comparing, and analyzing imagery acquired under different imaging conditions.

Chandrayaan-2 imagery can contain significant variations caused by:

- Different illumination conditions
- Changes in Sun angle
- Differences in image scale
- Viewing geometry
- Sensor characteristics
- Surface appearance
- Different spatial resolutions

These variations can make conventional feature matching and image registration unreliable.

MITRA addresses this through a structured registration pipeline in which correspondences are generated, filtered, geometrically verified, spatially analyzed, and finally used for image registration.

Rather than treating registration as a black-box operation, MITRA exposes intermediate correspondence and verification results so that the quality of the final result can be inspected and quantified.

---

# Problem Statement

**Smart India Hackathon 2026 — SIH26166**

The problem requires a system capable of performing multi-modal, Sun-angle and scale-invariant image correspondence using Chandrayaan-2 optical imagery.

The system should be capable of:

- Finding reliable correspondence points between images
- Handling changes in illumination and Sun angle
- Handling scale variations
- Handling viewpoint and viewing-geometry differences
- Producing robust correspondence points
- Maintaining a uniform spatial distribution of match points
- Registering the source image against the reference image
- Providing sub-pixel registration accuracy
- Quantitatively evaluating registration quality

MITRA is designed around these requirements.

---

# Objectives

The main objectives of MITRA are:

1. Establish reliable correspondences between source and reference imagery.
2. Improve correspondence robustness under imaging variations.
3. Remove geometrically inconsistent matches.
4. Encourage spatially distributed correspondence points.
5. Estimate the geometric transformation between images.
6. Register the source image with respect to the reference image.
7. Provide quantitative registration metrics.
8. Provide visual evidence for correspondence and registration quality.
9. Validate the system using relevant Chandrayaan-2 imagery.
10. Provide a foundation for evaluating advanced correspondence methods.

---

# MITRA Solution

MITRA treats image registration as a sequence of measurable stages rather than a single operation.

```text
                 SOURCE IMAGE
                      +
                 REFERENCE IMAGE
                      |
                      v
               PREPROCESSING
                      |
                      v
             FEATURE EXTRACTION
                      |
                      v
              FEATURE MATCHING
                      |
                      v
               RATIO TEST
                      |
                      v
          GEOMETRIC VERIFICATION
                 RANSAC
                      |
                      v
          SPATIAL DISTRIBUTION
                      |
                      v
          TRANSFORMATION ESTIMATION
                      |
                      v
             IMAGE REGISTRATION
                      |
                      v
              QUALITY ANALYSIS
                      |
          +-----------+-----------+
          |                       |
          v                       v
       METRICS              VISUALIZATION

Each stage contributes evidence to the final registration result.

Core Pipeline
1. Preprocessing

The source and reference images are prepared for feature extraction and correspondence.

Depending on the input data, preprocessing can include:

Grayscale conversion
Normalization
Resolution handling
Scaling
Noise handling
Image preparation for feature extraction

The exact preprocessing operations depend on the characteristics of the input imagery.

2. Feature Extraction

The baseline implementation uses SIFT (Scale-Invariant Feature Transform) to detect local image features.

SIFT is used because local features can provide correspondence candidates even when images differ in scale and rotation.

The output includes:

Keypoint locations
Feature descriptors
Local image structure information
3. Feature Matching

Descriptors extracted from the source and reference images are compared to identify candidate correspondences.

The current baseline uses:

BFMatcher — Brute Force Matcher

The result is a set of candidate feature correspondences.

4. Ratio Test

Candidate matches are filtered using Lowe's ratio test.

The ratio test helps remove ambiguous matches where the best and second-best descriptor matches are too similar.

Conceptually:

Candidate Matches
        |
        v
   Ratio Test
        |
   +----+----+
   |         |
Accepted   Rejected
5. Geometric Verification

Descriptor similarity alone does not guarantee that a correspondence is physically meaningful.

MITRA therefore applies RANSAC-based geometric verification.

RANSAC attempts to identify a transformation that is consistent with a large subset of the candidate correspondences.

The resulting correspondences are divided into:

Verified Inliers
       +
Rejected Outliers

The transformation estimated during this stage is used for subsequent registration.

6. Spatial Distribution

Feature matches may naturally cluster around visually distinctive regions.

For example:

Poor Distribution

+---------------------------+
|                           |
|                           |
|        X X X X X          |
|        X X X X X          |
|        X X X X X          |
|                           |
|                           |
+---------------------------+

A large number of matches does not necessarily mean that the entire image is well constrained.

MITRA therefore evaluates correspondence distribution spatially.

A better distribution may look like:

Better Distribution

+---------------------------+
| X                     X   |
|                           |
|       X           X       |
|                           |
|   X                   X   |
|                           |
| X                     X   |
+---------------------------+

The objective is to provide broader spatial support for the estimated transformation.

MITRA uses spatial filtering and coverage analysis to identify and retain a more distributed set of correspondences.

Why Spatial Distribution Matters

Transformation estimation depends not only on the number of correspondences but also on where those correspondences occur.

If hundreds of matches are concentrated in a small region, they may provide limited information about geometric distortion across the rest of the image.

Spatially distributed correspondences can provide stronger geometric support across the image.

MITRA therefore considers two complementary properties:

Correspondence Quality
        +
Spatial Distribution
        |
        v
Better Geometric Support

Spatial distribution is treated as an additional quality criterion rather than a replacement for geometric verification.

Supported Imagery

MITRA is designed around Chandrayaan-2 optical imagery.

The project considers relevant Chandrayaan-2 imaging payloads including:

OHRC

Orbiter High Resolution Camera.

TMC / TMC-2

Terrain Mapping Camera imagery.

IIRS

Imaging Infrared Spectrometer imagery.

Where available, metadata can be incorporated into the workflow, including:

Sensor
Acquisition information
Image dimensions
Spatial resolution
Sun geometry
Viewing conditions
Image scale

The exact metadata available depends on the source product.

System Architecture
                         MITRA
                           |
             +-------------+-------------+
             |                           |
        SOURCE IMAGE               REFERENCE IMAGE
             |                           |
             +-------------+-------------+
                           |
                           v
                    PREPROCESSING
                           |
                           v
                  FEATURE EXTRACTION
                           |
                           v
                   FEATURE MATCHING
                           |
                           v
                     RATIO TEST
                           |
                           v
                RANSAC VERIFICATION
                           |
                           v
               SPATIAL DISTRIBUTION
                           |
                           v
              TRANSFORMATION MODEL
                           |
                           v
                  IMAGE REGISTRATION
                           |
              +------------+------------+
              |                         |
              v                         v
           METRICS                VISUAL OUTPUT
              |                         |
              +------------+------------+
                           |
                           v
                    FINAL ANALYSIS
Registration Workflow
Step 1 — Select Source and Reference Images

The user selects:

Source image
Reference image

Associated metadata can be displayed to provide context for the registration run.

Step 2 — Inspect Image Characteristics

The system can expose relevant information such as:

Sensor
Resolution
Image dimensions
Acquisition information
Illumination information
Sun geometry

This helps the user understand the registration scenario before execution.

Step 3 — Preprocess

The images are prepared for correspondence extraction.

Step 4 — Extract Features

SIFT detects keypoints and generates descriptors for the source and reference images.

Step 5 — Generate Candidate Correspondences

Feature descriptors are matched to produce candidate correspondence pairs.

Step 6 — Filter Matches

The ratio test removes ambiguous candidate matches.

Step 7 — Geometric Verification

RANSAC identifies geometrically consistent correspondences and estimates the transformation model.

Step 8 — Analyze Spatial Distribution

The verified correspondences are evaluated according to their spatial distribution.

The system can calculate coverage and apply spatial filtering where appropriate.

Step 9 — Register the Source Image

The estimated transformation is applied to the source image.

The output is aligned with the reference image.

Step 10 — Evaluate the Result

The registration is evaluated using:

Inlier count
Inlier ratio
RMSE
Spatial coverage
Processing time
Transformation parameters

The system also provides visual correspondence and registration evidence.

Evaluation Metrics

MITRA uses multiple metrics because no single metric completely describes registration quality.

Inlier Count

The number of correspondences that remain geometrically consistent after RANSAC verification.

Inlier Count = Number of Geometrically Consistent Matches
Inlier Ratio

The ratio of geometrically verified correspondences to candidate correspondences.

Inlier Ratio = Inliers / Candidate Matches

A higher inlier ratio generally indicates that a larger proportion of candidate matches are geometrically consistent.

RMSE

Root Mean Square Error measures the reprojection or registration error associated with the estimated transformation.

Lower error generally indicates closer geometric agreement under the selected evaluation procedure.

Spatial Coverage

Spatial coverage measures how broadly the verified correspondences are distributed across the image.

This helps distinguish between:

Many concentrated matches

and

Fewer but spatially distributed matches
Processing Time

The system records processing time to evaluate computational efficiency.

This is particularly relevant when comparing classical and more computationally intensive correspondence methods.

Baseline and MITRA Comparison

MITRA is designed to be evaluated against established baseline registration approaches.

The benchmark structure is:

Metric	Baseline	MITRA
Candidate Matches	To be measured	To be measured
Inliers	To be measured	To be measured
Inlier Ratio	To be measured	To be measured
RMSE	To be measured	To be measured
Spatial Coverage	To be measured	To be measured
Processing Time	To be measured	To be measured

Actual values will be added after experiments on real and representative datasets.

No benchmark values are fabricated during development.

Frontend

MITRA provides a scientific workstation-style interface designed around the actual image registration workflow.

The interface includes:

Dashboard

Provides an overview of registration activity and key system metrics.

Image Registration

The main workspace for:

Selecting source and reference images
Inspecting metadata
Running registration
Viewing the processing pipeline
Inspecting correspondence information
Analysis and Results

Provides detailed evidence from a registration run.

This includes:

Source/reference comparison
Correspondence visualization
Inlier visualization
Inlier ratio
RMSE
Spatial coverage
Transformation information
Registered image
Overlay comparison
Dataset

Provides an interface for browsing available imagery and associated metadata.

Methodology

Documents the registration methodology and explains the different stages of the MITRA pipeline.

Mission Information

Provides contextual information about Chandrayaan-2 and the relevant imaging payloads.

Technology Stack
Frontend
Next.js
React
TypeScript
Tailwind CSS
Backend and Computer Vision
Python
OpenCV
SIFT
BFMatcher
RANSAC
Image transformation techniques
Visualization
React
Image overlays
Correspondence visualization
Registration comparison
Quantitative metric visualization
Project Structure
MITRA/
│
├── frontend/
│   ├── app/
│   ├── components/
│   ├── lib/
│   └── public/
│
├── backend/
│   ├── app/
│   │   └── cv/
│   └── tests/
│
├── data/
│
├── scripts/
│
├── docs/
│
└── README.md

The structure may change as the backend and real-data pipeline evolve.

Computer Vision Components

The current baseline implementation contains modules for:

Feature Detection
       |
       v
Descriptor Extraction
       |
       v
Descriptor Matching
       |
       v
Ratio Test
       |
       v
RANSAC
       |
       v
Transformation Estimation
       |
       v
Spatial Filtering
       |
       v
Registration
       |
       v
Metrics

The modular design allows individual stages to be replaced or improved without redesigning the entire system.

Data

MITRA uses imagery and metadata relevant to the Chandrayaan-2 mission.

During development, synthetic lunar-analogue imagery may be used for controlled testing of:

Feature matching
Registration
Transformation estimation
UI visualization
Spatial filtering
Metric calculations

Synthetic data is explicitly separated from mission data.

Synthetic experiments are not used as evidence of real Chandrayaan-2 performance.

Current Development Status

MITRA is currently under active development.

Completed
Registration workstation interface
Source/reference image workflow
Image metadata presentation
Feature extraction pipeline
SIFT-based matching
Brute-force descriptor matching
Ratio-test filtering
RANSAC-based geometric verification
Transformation estimation
Registration visualization
Spatial correspondence analysis
Grid-based spatial filtering
Coverage measurement
Quantitative metric interface
Methodology interface
Mission information interface
Synthetic development imagery
Initial computer vision testing
In Progress
Real Chandrayaan-2 data integration
Mission metadata integration
End-to-end frontend/backend integration
Real-data registration validation
Representative benchmark creation
Baseline comparison
Evaluation across different imaging conditions
Planned
OHRC evaluation
TMC evaluation
Applicable IIRS evaluation
Cross-sensor experiments
Illumination variation testing
Scale variation testing
Viewpoint variation testing
Advanced correspondence methods
Performance optimization
Roadmap
Phase 1 — Core Registration
 Feature extraction
 Descriptor matching
 Ratio-test filtering
 RANSAC geometric verification
 Transformation estimation
 Image registration
 Basic metrics
Phase 2 — Spatial Correspondence
 Spatial distribution analysis
 Grid-based filtering
 Spatial coverage measurement
 Extensive real-data validation
Phase 3 — Chandrayaan-2 Validation
 Integrate real OHRC products
 Integrate real TMC products
 Evaluate applicable IIRS imagery
 Create representative image pairs
 Test illumination variations
 Test scale variations
 Test viewing geometry variations
 Establish baseline benchmark
 Establish MITRA benchmark
 Compare results quantitatively
Phase 4 — Advanced Correspondence
 Evaluate learned feature extractors
 Evaluate learned feature matching
 Evaluate modern correspondence architectures
 Evaluate dense correspondence methods
 Compare classical and learned approaches
 Optimize accuracy and runtime
Advanced Methods

The current system establishes a classical computer vision baseline before introducing more computationally expensive approaches.

Potential future methods include:

SuperPoint
LightGlue
LoFTR
Other learned correspondence methods

These approaches will be evaluated experimentally.

They will not be added simply for the purpose of increasing model complexity.

The objective is to determine whether advanced methods provide measurable improvements for the specific characteristics of Chandrayaan-2 imagery.

Design Philosophy

MITRA follows three core principles.

1. Reliable Correspondence First

A registration result depends on the quality of the correspondences supporting it.

2. Registration Second

Geometric transformation and image alignment are performed after correspondence generation and verification.

3. Evidence Throughout

The system exposes:

Candidate matches
Verified inliers
Spatial distribution
Transformation parameters
Registration error
Registered imagery
Quantitative metrics

This makes it possible to inspect the reasoning behind a registration result instead of relying only on a final image.

Scientific Approach

MITRA does not attempt to claim that feature-based image registration or RANSAC are new algorithms.

Instead, the project focuses on building a practical and transparent registration workflow around the specific requirements of Chandrayaan-2 imagery.

The system combines:

Established Computer Vision
          +
Chandrayaan-2 Data Characteristics
          +
Spatial Correspondence Analysis
          +
Quantitative Evaluation
          |
          v
     MITRA Pipeline

The effectiveness of the system will be established through experiments rather than assumptions.

Limitations

MITRA is currently under development and has several limitations.

Data Dependency

Registration performance depends on the characteristics and quality of the available imagery.

Appearance Variations

Classical feature methods can degrade under severe illumination, appearance, or sensor differences.

Cross-Sensor Registration

Correspondence between substantially different sensors can be more difficult than same-sensor registration.

Spatial Filtering

Spatial distribution alone does not guarantee correct correspondence or accurate registration.

Metric Interpretation

Individual metrics should not be interpreted independently.

For example, a high inlier count does not automatically imply a high-quality registration if the correspondences are heavily clustered.

Benchmarking

Meaningful performance claims require evaluation on representative real mission imagery.

Future Work

Future development will focus on improving robustness while preserving measurable and interpretable results.

Potential research and engineering directions include:

Multi-scale correspondence
Learned local features
Learned feature matching
Dense correspondence
Illumination normalization
Sensor-specific preprocessing
Improved geometric models
Robust transformation estimation
Automated parameter selection
Large-scale benchmark evaluation
Runtime optimization
Hardware-aware processing
Installation
Prerequisites

Recommended environment:

Node.js
npm
Python 3.x
pip
OpenCV

Verify the installations:

node --version
npm --version
python --version
pip --version
Frontend Setup

Clone the repository:

git clone <REPOSITORY_URL>

Move into the project directory:

cd MITRA

Install frontend dependencies:

npm install

Start the development server:

npm run dev

The application can then be accessed through the local development URL displayed by Next.js.

Backend Setup

Create a Python virtual environment:

python -m venv .venv

Activate it on Windows:

.venv\Scripts\activate

Activate it on Linux/macOS:

source .venv/bin/activate

Install the required dependencies:

pip install -r requirements.txt

Run the backend according to the project's backend entry point.

Usage

A typical MITRA registration run follows:

1. Select Source Image
2. Select Reference Image
3. Inspect Metadata
4. Start Registration
5. Extract Features
6. Generate Candidate Matches
7. Apply Ratio Test
8. Perform RANSAC Verification
9. Analyze Spatial Distribution
10. Estimate Transformation
11. Register Source Image
12. Inspect Metrics
13. Inspect Correspondences
14. Compare Registered and Reference Images

The resulting run should provide both visual and quantitative evidence.

Example Output

A successful registration run can provide information such as:

Source Image:       Image A
Reference Image:    Image B

Candidate Matches:  N
Verified Inliers:   N
Inlier Ratio:       N %

RMSE:               N px
Spatial Coverage:   N %
Processing Time:    N ms

Transformation:
[ estimated transformation matrix ]

The values above are placeholders and are not intended to represent benchmark results.

Actual values should be generated from the executed registration pipeline.

Testing

The computer vision pipeline should be tested using:

Controlled synthetic transformations
Same-sensor image pairs
Different imaging conditions
Challenging correspondence cases
Insufficient-feature cases
Incorrect or incompatible image pairs

Tests should verify:

Feature extraction
Matching
Ratio filtering
RANSAC behavior
Transformation estimation
Spatial filtering
Registration
Metric calculation
Failure handling
Research and Evaluation

The final evaluation will focus on representative image pairs rather than a single successful example.

The benchmark should include cases covering:

Same Sensor
     |
     +-- Different Illumination
     +-- Different Scale
     +-- Different Viewing Geometry
     |
Cross Sensor
     |
     +-- Applicable Multi-Modal Cases

The evaluation should compare the baseline pipeline against MITRA using consistent metrics and datasets.

Results should be reported with enough information to understand:

Which image pair was tested
Which sensors were involved
What imaging conditions existed
Which method was used
How many matches were generated
How many survived geometric verification
What the inlier ratio was
What registration error was obtained
How correspondences were spatially distributed
How long the method required
Reproducibility

MITRA aims to make registration experiments reproducible.

A registration experiment should record:

Input Images
      |
      +-- Image Metadata
      |
      +-- Algorithm
      |
      +-- Parameters
      |
      +-- Candidate Matches
      |
      +-- Inliers
      |
      +-- Transformation
      |
      +-- RMSE
      |
      +-- Spatial Coverage
      |
      +-- Processing Time
      |
      +-- Registered Output

This allows different methods and parameter configurations to be compared consistently.

Project Principles

MITRA follows several development principles:

Do Not Fabricate Results

Benchmark numbers are only reported after actual experiments.

Prefer Evidence Over Claims

Visual and quantitative evidence should support technical claims.

Keep the Pipeline Interpretable

Intermediate correspondence and geometric results should remain inspectable.

Use Complexity Only When It Helps

Advanced models should be introduced when they provide measurable improvements.

Validate on Real Data

Synthetic data is useful for controlled testing but does not replace validation on real Chandrayaan-2 imagery.

Smart India Hackathon 2026

MITRA is being developed for:

Smart India Hackathon 2026

Problem Statement: SIH26166

Domain: Space Technology

The project addresses the challenge of multi-modal, Sun-angle and scale-invariant image correspondence using Chandrayaan-2 optical imagery.

Expected Outcome

The intended outcome of MITRA is a working image correspondence and registration system capable of:

Chandrayaan-2 Image Pair
          |
          v
Reliable Correspondences
          |
          v
Geometric Verification
          |
          v
Spatially Distributed Matches
          |
          v
Accurate Registration
          |
          v
Quantitative Evaluation
          |
          v
Registered Image + Evidence

The final system will be evaluated based on actual experimental performance on relevant imagery.

Team

MITRA is developed as a team project for Smart India Hackathon 2026.

The project combines:

Computer Vision
Remote Sensing
Image Processing
Data Engineering
Full-Stack Development
Research
System Integration
Acknowledgement

MITRA is developed as an academic and engineering project for Smart India Hackathon 2026, based on the requirements of problem statement SIH26166.

The project is intended for research, experimentation, and demonstration purposes.

Disclaimer

MITRA is a student-developed project.

It is not an official ISRO software system, nor does it represent an official Chandrayaan-2 processing pipeline.

Any references to Chandrayaan-2 payloads, datasets, or mission information are used in the context of the stated problem and project development.

Synthetic imagery used during development is not Chandrayaan-2 mission data and is not used as evidence of real mission-data performance.

All performance claims and benchmark comparisons will be based on actual experimental results.

License

This project is currently under development.

License information will be added when the project is finalized.
