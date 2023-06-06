# PeachDB

<h3 align="center"><strong>PeachDB - the AI-First, Embeddings Database</strong></h3>
<h4 align="center">Build memory for your AI products in <strong>minutes!</strong></h4>

<br/>

**Our core API has 4 functions**

```python
from peachdb import PeachDB

# Create a new PeachDB instance or reference an existing one
db = PeachDB(
    project_name="my_app",
    embedding_generator="imagebind",  # "imagebind" or "sentence_transformer_L12"
    embedding_backend="exact_cpu",  # "exact_cpu", "exact_gpu", or "approx"
    distance_metric="cosine",  # "cosine" or "l2"
)

# Auto-compute & upsert embeddings at scale using the specified `embedding_generator` model on Modal
db.upsert_text(  # or "upsert_image" or "upsert_audio"
    csv_path="/path/to/local/csv",  # or "s3://path/to/csv"
    column_to_embed="foo",  # column values can either be string or public URI to image/audio
    id_column_name="id",
    embeddings_output_s3_bucket_uri=None,  # required when using S3 URI for `csv_path`
    max_rows=None,  # or N to process N rows
)

# Query top 5 similar results
ids, distances, results_df = db.query(
    query_input='An example query',  # or path to an image/audio file
    modality='text',  # "text", "image", or "audio"
    top_k=5
)

# Deploy database as a publicly accessible FastAPI server
# GET /query?query_input='An example query'&modality=[text|image|audio]&top_k=5 to fetch 5 most similar results
db.deploy()
```

## Why another embedding database?
We've streamlined the entire end-to-end process of creating, storing, and retrieving embeddings, making it developer-friendly, seamless, and cost-effective. You no longer have to build custom pipelines or fret over hardware setups & scalability issues. PeachDB ensures you can get started within minutes, leaving the worries of cost optimization and scale to us.

Our key features include:
* **Automated, cost-effective & large-scale embedding computation**: We leverage serverless GPU functions (through [Modal](https://modal.com/)) to compute embeddings on a large scale efficiently and affordably.
    - For instance, we processed the [Kaggle 5M song lyrics dataset](https://www.kaggle.com/datasets/nikhilnayak123/5-million-song-lyrics-dataset?resource=download&select=ds2.csv) in just *12 minutes at a cost of $4.90*, using sentence transformers!
    - We've developed Modal wrappers for:
        - [Sentence Transformer L12](https://huggingface.co/sentence-transformers/all-MiniLM-L12-v2)
        - [ImageBind](https://github.com/facebookresearch/ImageBind),
        - *Coming soon*, support for:
            - Open-source embedding models such as [OpenCLIP](https://github.com/mlfoundations/open_clip), [Microsoft E5-v2](https://arxiv.org/pdf/2212.03533.pdf), and more.
            - Multi-threaded [OpenAI](https://platform.openai.com/docs/guides/embeddings) embedding calculation.
    - *Coming soon*: Bring your own embeddings.
    - *Coming soon*: Custom embedding functions for even more flexibility.
* **Multimodality**: Native support for data with mixture of modalities (such as image/audio/video).
* **Highly Customizable**: PeachDB allows you to tailor its features to your needs. You can customize:
    - Embedding computation: as described above.
    - Backend: choose between `exact_cpu` (numpy), `exact_gpu` (torch), or `approx` (HNSW).
    - Distance metrics: `cosine` or `l2`.
* **Effortless Deployment**: Deploy PeachDB as a publicly available server with a single API. No need to worry about nginx or SSL certs.
    - *Coming soon*: Managed, scalable deployment.
* **Consistent API**: Experience the same API across all environments - dev, test, and prod.
* **Open Source**: Apache 2.0.


## Example

Below is a walkthrough of creating a web server for a music recommendation app. To power the app, we are using the [Kaggle 5M song lyric dataset](https://www.kaggle.com/datasets/nikhilnayak123/5-million-song-lyrics-dataset?resource=download&select=ds2.csv)


- Ssh into your remote instance (doesn't need GPU)
- Create & activate a new conda environment `conda create -n spoti_vibe python=3.10` & `conda activate spoti_vibe`
- Install PeachDB: `pip install peachdb`
- Setup [Modal](https://modal.com)
    - Create an account at [modal.com](https://modal.com)
    - Install the modal-client package: `pip install modal-client`
    - Setup token: `modal token new`

- (optional: for AWS S3) PeachDB accepts local & S3 paths to datasets for embedding computation. To use S3 URIs, ensure you've installed the `aws` cli and run `aws configure`. The credentials should have read & write access to the relevant bucket you plan to use.
- `mkdir spoti_vibe/`
- Create a new module inside the directory `server.py`
- Add the following code
    ```python
    from peachdb import PeachDB

    import os

    # Fetch the username & key by creating a new API token at https://www.kaggle.com/settings
    os.environ["KAGGLE_USERNAME"] = None  # set user name
    os.environ["KAGGLE_KEY"] = None  # set key

    import kaggle  # make sure you've run `pip install kaggle`

    kaggle.api.authenticate()
    # It can take a few mins to download depending on the network speed
    kaggle.api.dataset_download_files("nikhilnayak123/5-million-song-lyrics-dataset", path=".", unzip=True)

    db = PeachDB(
        project_name="spoti_vibe",
        distance_metric="cosine",
        embedding_backend="exact_cpu",
        embedding_generator="sentence_transformer_L12",
    )
    db.upsert_text(
        csv_path="./ds2.csv",  # dataset name as observed on Kaggle
        column_to_embed="lyrics",
        id_column_name="id",
    )

    db.deploy()  # Public URL will be printed to console
    ```

And that's it! You should now have a publicly available server that can listen to query requests from the user on: <br/>
`GET <PUBLIC_URL>/query?text='Happy, upbeat summer'&top_k=5`

## Use-cases
- Build web apps like - [clip.audio](https://www.clip.audio/) & [awesome-movies.life](https://awesome-movies.life/)
- Build ChatGPT for X
- Build ChatGPT plugins


## FAQs
Q) How can I delete a project?
Run `db.delete(project_name="my_app")`

## Get Involved
We welcome PR contributors and ideas for how to improve the project.

## License
[Apache 2.0](./LICENSE)
