# Analysis of Textimager Client
First attempt tried: ***Using the textimager client class to invoke the analysis engine on the server***.

Outcome: The class does only provide methods to use the preconfigured annotators when processing one jCas Element. Therefore this method did not work.

Second attempt: ***Using the textimager client class with processCollection to invoke Big Data analysis on custom engines***

The function family `processCollection` does take an `AnalysisEngineDescription` as parameter. So this may work to provide the custom AnalysisEngine to the servers.

Outcome: The line 308 in TextImager client contains a statement which yields a null pointer exception when invoked on aggregate pipelines. This statement does only work when invoked on 1 AnalysisEngine therefore one can only provide a primitive AnalysisEngine to this method as opposed to a whole pipeline.


Third attempt: ***Going one method deeper and analysing getUimaAsEngine to allow custom configured annotators.***

The function `getUimaAsEngine` is called by every process method in the TextImagerClient. Therefore it must contain the logic to create the Uima Engine on the servers. Placing a `System.exit(0)` right before `uimaAsEngine.deploy(deployFile.getAbsolutePath(), clientCtx);` and commenting out `deployFile.deleteOnExit();` yields the deploy file. This deploy file does **not** contain any configuration of the pipeline it just has a list of webservices to invoke. And a set of basic parameters as opposed to a full Annotator configuration.

Forth attempt: ***Investigating the analyse function of DUCCAPI.***

Could not get this to work. As one can see in [github fork](https://github.com/ShadowItaly/textimager-client). The function analyse does not get called if I process a directory or a file from the command line. When having a look at this [attempt](https://github.com/ShadowItaly/textimager-client/blob/ad99cfe83dcd081793f963b1fe0d27d1e5a6b232/src/main/java/org/hucompute/textimager/client/TextImagerClient.java#L769) even when invoking the method directly with a collection reader it refuses to call analyse. Further analysis showed that the function only gets called by the Rest class. I could not find any further references or uses of the rest class in the source code, except its definition. The class `DUCCAPI` get included by one other class and this is `SSHUtils` this class only uses the reference to get some constant variables.

The includes of DUCCAPI listed:
1. ` ..ager/client/rest/REST.java:31:1:import org.hucompute.textimager.client.rest.ducc.DUCCAPI;` uses the DUCCAPI but is not used anywhere in the codebase.
2. `..timager/util/SSHUtils.java:13:1:import org.hucompute.textimager.client.rest.ducc.DUCCAPI;` uses the DUCCAPI but only the constants, does not call **any** function of DUCCAPI.
3. `../textimager/client/Tmp.java:5:1:import org.hucompute.textimager.client.rest.ducc.DUCCAPI;` Is just a test class and does not really do anything with DUCCAPI.

The includes of REST class listed, since this is the only class to utilize DUCC API:
1. `--/-- None`

I tried to find references by doing a full text Grep of the code base by `import org.hucompute.textimager.client.rest.REST;`. I did the same for references of DUCCAPI e. g. `import org.org.hucompute.textimager.client.rest.ducc.DUCCAPI`.
