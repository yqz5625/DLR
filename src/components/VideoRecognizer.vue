<script setup lang="ts">
import { onMounted, onUnmounted, ref, type Ref } from "vue";
import { MultiFrameResultCrossFilter } from "dynamsoft-utility";
import { CameraEnhancer, CameraView } from "dynamsoft-camera-enhancer";
import {
  LabelRecognizerModule,
  type RecognizedTextLinesResult,
} from "dynamsoft-label-recognizer";
import {
  CapturedResultReceiver,
  CaptureVisionRouter,
} from "dynamsoft-capture-vision-router";

const uiContainer: Ref<HTMLElement | null> = ref(null);
const resultsContainer: Ref<HTMLElement | null> = ref(null);

const pInit: Ref<Promise<{
  cameraView: CameraView;
  cameraEnhancer: CameraEnhancer;
  router: CaptureVisionRouter;
}> | null> = ref(null);

const setting = {
  CaptureVisionTemplates: [
    {
      ImageROIProcessingNameArray: ["roi_default"],
      ImageSource: "",
      MaxParallelTasks: 4,
      MinImageCaptureInterval: 0,
      Name: "Default",
      OutputOriginalImage: 0,
      SemanticProcessingNameArray: null,
      Timeout: 10000,
    },
  ],
  CharacterModelOptions: [
    { DirectoryPath: "", FilterFilePath: "", Name: "NumberLetter" },
  ],
  GlobalParameter: { MaxTotalImageDimension: 0 },
  ImageParameterOptions: [
    {
      BaseImageParameterName: "",
      BinarizationModes: [
        {
          BinarizationThreshold: -1,
          BlockSizeX: 5,
          BlockSizeY: 5,
          EnableFillBinaryVacancy: 1,
          GrayscaleEnhancementModesIndex: -1,
          Mode: "BM_LOCAL_BLOCK",
          MorphOperation: "Close",
          MorphOperationKernelSizeX: -1,
          MorphOperationKernelSizeY: -1,
          MorphShape: "Rectangle",
          ThresholdCompensation: 8,
        },
      ],
      ColourConversionModes: [
        {
          BlueChannelWeight: -1,
          GreenChannelWeight: -1,
          Mode: "CICM_GENERAL",
          RedChannelWeight: -1,
          ReferChannel: "H_CHANNEL",
        },
      ],
      GrayscaleEnhancementModes: [
        {
          Mode: "GEM_GENERAL",
          Sensitivity: -1,
          SharpenBlockSizeX: -1,
          SharpenBlockSizeY: -1,
          SmoothBlockSizeX: -1,
          SmoothBlockSizeY: -1,
        },
      ],
      GrayscaleTransformationModes: [
        { Mode: "GTM_ORIGINAL" },
        { Mode: "GTM_INVERTED" },
      ],
      IfEraseTextZone: 0,
      Name: "ip_dlrDefault",
      RegionPredetectionModes: [
        {
          AspectRatioRange: "[]",
          FindAccurateBoundary: 0,
          ForeAndBackgroundColours: "[]",
          HeightRange: "[]",
          ImageParameterName: "",
          MeasuredByPercentage: 1,
          MinImageDimension: 262144,
          Mode: "RPM_GENERAL",
          RelativeRegions: "[]",
          Sensitivity: 1,
          SpatialIndexBlockSize: 5,
          WidthRange: "[]",
        },
      ],
      ScaleDownThreshold: 2300,
      ScaleUpModes: [
        {
          AcuteAngleWithXThreshold: -1,
          LetterHeightThreshold: 0,
          Mode: "SUM_AUTO",
          ModuleSizeThreshold: 0,
          TargetLetterHeight: 0,
          TargetModuleSize: 0,
        },
      ],
      TextDetectionMode: {
        CharHeightRange: [20, 1000, 1],
        Direction: "HORIZONTAL",
        MaxSpacingInALine: -1,
        Mode: "TTDM_LINE",
        Sensitivity: 7,
      },
      TextureDetectionModes: [
        { Mode: "TDM_GENERAL_WIDTH_CONCENTRATION", Sensitivity: 5 },
      ],
    },
  ],
  LabelRecognizerTaskSettingOptions: [
    {
      BaseLabelRecognizerTaskSettingName: "",
      DictionaryCorrectionThresholds: [
        { MaxWordLength: 256, MinWordLength: 3, Threshold: 1 },
      ],
      DictionaryPath: "",
      MaxThreadsInOneTask: 4,
      Name: "dlr_task_default",
      SectionImageParameterArray: [
        {
          ContinueWhenPartialResultsGenerated: 1,
          ImageParameterName: "ip_dlrDefault",
          Section: "ST_REGION_PREDETECTION",
        },
        {
          ContinueWhenPartialResultsGenerated: 1,
          ImageParameterName: "ip_dlrDefault",
          Section: "ST_TEXT_LINE_LOCALIZATION",
        },
        {
          ContinueWhenPartialResultsGenerated: 1,
          ImageParameterName: "ip_dlrDefault",
          Section: "ST_TEXT_LINE_RECOGNITION",
        },
      ],
      StartSection: "ST_REGION_PREDETECTION",
      StringLengthRange: [3, 10],
      StringRegExPattern: "^\d{6}$",
      TerminateSetting: { Section: "ST_NULL", Stage: "IRUT_NULL" },
      TextLineSpecificationNameArray: ["tls_default"],
    },
  ],
  TargetROIDefOptions: [
    {
      BaseTargetROIDefName: "",
      Location: {},
      Name: "roi_default",
      PauseFlag: 0,
      TaskSettingNameArray: ["dlr_task_default"],
    },
  ],
  TextLineSpecificationOptions: [
    {
      ApplicableTextLineNumbers: "",
      BaseTextLineSpecificationName: "",
      BinarizationModes: [
        {
          BinarizationThreshold: -1,
          BlockSizeX: 5,
          BlockSizeY: 5,
          EnableFillBinaryVacancy: 1,
          GrayscaleEnhancementModesIndex: -1,
          Mode: "BM_LOCAL_BLOCK",
          MorphOperation: "Erode",
          MorphOperationKernelSizeX: -1,
          MorphOperationKernelSizeY: -1,
          MorphShape: "Rectangle",
          ThresholdCompensation: 10,
        },
      ],
      CharHeightRange: [20, 1000, 1],
      CharacterModelName: "NumberLetter",
      CharacterNormalizationModes: [
        { Mode: "CNM_AUTO", MorphArgument: "", MorphOperation: "Erode" },
      ],
      GrayscaleEnhancementModes: [
        {
          Mode: "GEM_GENERAL",
          Sensitivity: -1,
          SharpenBlockSizeX: -1,
          SharpenBlockSizeY: -1,
          SmoothBlockSizeX: -1,
          SmoothBlockSizeY: -1,
        },
      ],
      Name: "tls_default",
      StringLengthRange: [5, 7],
      StringRegExPattern: "[0-9]{6}",
    },
  ],
};

const pVal = ref(0)
const init = async (): Promise<{
  cameraView: CameraView;
  cameraEnhancer: CameraEnhancer;
  router: CaptureVisionRouter;
}> => {
  try {
    LabelRecognizerModule.onDataLoadProgressChanged = (
      filePath: string,
      tag: string,
      progress: { loaded: number; total: number }
    ) => {
      if (tag === "starting") {
        console.log("load started...");
      } else if (tag === "completed") {
        console.log("load ended...");
      } else {
        console.log(
          "Loading resources progress: " +
            progress!.loaded +
            "/" +
            progress!.total
        );
      }
    };


    // Create a `CameraEnhancer` instance for camera control and a `CameraView` instance for UI control.
    const cameraView = await CameraView.createInstance();
    const cameraEnhancer = await CameraEnhancer.createInstance(cameraView);

    let scanRegion = {
      x: 30,
      y: 50,
      width: 40,
      height: 10,
      isMeasuredInPercentage: true,
    };
    cameraEnhancer.setScanRegion(scanRegion);

    uiContainer.value!.append(cameraView.getUIElement()); // Get default UI and append it to DOM.

    // Create a `CaptureVisionRouter` instance and set `CameraEnhancer` instance as its image source.
    const router = await CaptureVisionRouter.createInstance();
    router.setInput(cameraEnhancer);
    await router.initSettings(setting);

    // Define a callback for results.
    const resultReceiver = new CapturedResultReceiver();
    resultReceiver.onRecognizedTextLinesReceived = (
      result: RecognizedTextLinesResult
    ) => {
      if (!result.textLineResultItems.length) return;

      resultsContainer.value!.innerHTML = "";
      console.log(result);
      for (let item of result.textLineResultItems) {
        if (item.confidence > 70) {
          resultsContainer.value!.innerHTML += `${item.text}<br><hr>`;
        }
      }
    };
    router.addResultReceiver(resultReceiver);

    // Filter out unchecked and duplicate results.
    const filter = new MultiFrameResultCrossFilter();
    filter.enableResultCrossVerification("text_line", true); // Filter out unchecked text.
    // Filter out duplicate text lines within 3 seconds.
    filter.enableResultDeduplication("text_line", true);
    filter.setDuplicateForgetTime("text_line", 3000);
    await router.addResultFilter(filter);

    // Open camera and start scanning text.
    await cameraEnhancer.open();
    await router.startCapturing("Default");
    return {
      cameraView,
      cameraEnhancer,
      router,
    };
  } catch (ex: any) {
    let errMsg = ex.message || ex;
    console.error(errMsg);
    alert(errMsg);
    throw ex;
  }
};

onMounted(async () => {
  pInit.value = init();
});

onUnmounted(async () => {
  if (pInit.value) {
    const { cameraView, cameraEnhancer, router } = await pInit.value;
    router.dispose();
    cameraEnhancer.dispose();
    cameraView.dispose();
  }
  console.log("VideoCapture Component Unmount");
});
</script>

<template>
  <div>
    <div ref="uiContainer" class="div-ui-container"></div>
    Results:
    <br />
    <div ref="resultsContainer" class="div-results-container"></div>
  </div>
</template>

<style scoped>
.div-ui-container {
  width: 100%;
  height: 70vh;
  background: #eee;
}

.div-results-container {
  width: 100%;
  height: 10vh;
  overflow: auto;
}
</style>
