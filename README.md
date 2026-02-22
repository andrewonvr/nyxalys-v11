using BepInEx;
using ExitGames.Client.Photon;
using GorillaExtensions;
using GorillaLocomotion;
using HarmonyLib;
using Photon.Pun;
using StupidTemplate.Classes;
using StupidTemplate.Menu;
using StupidTemplate.Notifications;
using System;
using System.Collections.Generic;
using System.Diagnostics.Contracts;
using UnityEngine;
using UnityEngine.InputSystem;
using UnityEngine.XR;
using static StupidTemplate.Menu.Main;

namespace StupidTemplate.Mods
{
    public class Movement
    {
        public static void Fly()
        {
            if (ControllerInputPoller.instance.rightControllerPrimaryButton)
            {
                GTPlayer.Instance.transform.position += GorillaTagger.Instance.headCollider.transform.forward * Time.deltaTime * Settings.Movement.flySpeed;
                GorillaTagger.Instance.rigidbody.linearVelocity = Vector3.zero;
            }
        }

        public static void FlyTrigger()
        {
            if (ControllerInputPoller.instance.rightControllerIndexFloat > 0.5)
            {
                GTPlayer.Instance.transform.position += GorillaTagger.Instance.headCollider.transform.forward * Time.deltaTime * Settings.Movement.flySpeed;
                GorillaTagger.Instance.rigidbody.linearVelocity = Vector3.zero;
            }
        }

        public static void SpazHands()
        {
            if (ControllerInputPoller.instance.rightControllerPrimaryButton)
            {
                VRRig.LocalRig.enabled = false;

                VRRig.LocalRig.transform.position = GorillaTagger.Instance.bodyCollider.transform.position + new Vector3(0f, 0.15f, 0f);
                VRRig.LocalRig.transform.rotation = GorillaTagger.Instance.bodyCollider.transform.rotation;

                VRRig.LocalRig.head.rigTarget.transform.rotation = GorillaTagger.Instance.headCollider.transform.rotation;

                VRRig.LocalRig.leftHand.rigTarget.transform.rotation = RandomQuaternion();
                VRRig.LocalRig.rightHand.rigTarget.transform.rotation = RandomQuaternion();

                VRRig.LocalRig.leftHand.rigTarget.transform.position = GorillaTagger.Instance.leftHandTransform.position + VRRig.LocalRig.leftHand.rigTarget.transform.forward * 3f;
                VRRig.LocalRig.rightHand.rigTarget.transform.position = GorillaTagger.Instance.rightHandTransform.position + VRRig.LocalRig.rightHand.rigTarget.transform.forward * 3f;
            }
            else

                VRRig.LocalRig.enabled = true;
        }

        static GameObject PlatL, PlatR;
        private static object cc;

        public static void Platforms()
        {
            if (ControllerInputPoller.instance.leftGrab)
            {
                if (PlatL == null)
                {
                    PlatL = GameObject.CreatePrimitive(PrimitiveType.Cube);
                    PlatL.transform.localScale = new Vector3(0.04f, 0.28f, 0.35f);
                    PlatL.transform.position = GorillaTagger.Instance.leftHandTransform.position + new Vector3(0f, -0.06f, 0f);
                    PlatL.transform.rotation = GorillaTagger.Instance.leftHandTransform.rotation;
                    PlatL.GetComponent<Renderer>().material.shader = Shader.Find("GorillaTag/UberShader");
                    PlatL.GetComponent<Renderer>().material.color = StupidTemplate.Settings.backgroundColor.GetCurrentColor();
                    ColorChanger.cc = PlatL.AddComponent<ColorChanger>(); // 
                }
            }
            else
            {
                if (PlatL != null)
                {
                    Rigidbody rb = PlatL.AddComponent(typeof(Rigidbody)) as Rigidbody;
                    rb.velocity = GorillaLocomotion.GTPlayer.Instance.GetHandVelocityTracker(true).GetAverageVelocity(true, 0);
                    GameObject.Destroy(PlatL, 0.1f);
                    PlatL = null;
                }
            }

            if (ControllerInputPoller.instance.rightGrab)
            {
                if (PlatR == null)
                {
                    PlatR = GameObject.CreatePrimitive(PrimitiveType.Cube);
                    PlatR.transform.localScale = new Vector3(0.04f, 0.28f, 0.35f);
                    PlatR.transform.position = GorillaTagger.Instance.rightHandTransform.position + new Vector3(0f, -0.06f, 0f);
                    PlatR.transform.rotation = GorillaTagger.Instance.rightHandTransform.rotation;
                    PlatR.GetComponent<Renderer>().material.shader = Shader.Find("GorillaTag/UberShader");
                    PlatR.GetComponent<Renderer>().material.color = StupidTemplate.Settings.backgroundColor.GetCurrentColor();
                    ColorChanger.cc = PlatR.AddComponent<ColorChanger>();
                }
            }
            else
            {
                if (PlatL != null)
                {
                    Rigidbody rb = PlatR.AddComponent(typeof(Rigidbody)) as Rigidbody;
                    rb.velocity = GorillaLocomotion.GTPlayer.Instance.GetHandVelocityTracker(true).GetAverageVelocity(true, 0);
                    GameObject.Destroy(PlatR, 0.1f);
                    PlatR = null;
                }
            }
        }

        public static bool previousTeleportTrigger;
        public static void TeleportGun()
        {
            if (ControllerInputPoller.instance.rightGrab)
            {
                var GunData = RenderGun();
                GameObject NewPointer = GunData.NewPointer;

                if (ControllerInputPoller.TriggerFloat(XRNode.RightHand) > 0.5f && !previousTeleportTrigger)
                {
                    GTPlayer.Instance.TeleportTo(NewPointer.transform.position + Vector3.up, GTPlayer.Instance.transform.rotation);
                    GorillaTagger.Instance.rigidbody.linearVelocity = Vector3.zero;
                }

                previousTeleportTrigger = ControllerInputPoller.TriggerFloat(XRNode.RightHand) > 0.5f;
            }
        }



        private static float delaybetweenscore;
        public static void MaxQuestScore()
        {
            if (Time.time > delaybetweenscore)
            {
                delaybetweenscore = Time.time + 1f;
                VRRig.LocalRig.SetQuestScore(int.MaxValue);
            }
        }

        public static int targetQuestScore = 999;
        public static void CustomQuestScore()
        {
            if (Time.time > delaybetweenscore)
            {
                delaybetweenscore = Time.time + 1f;
                VRRig.LocalRig.SetQuestScore(targetQuestScore);
            }
        }

        [HarmonyPatch(typeof(VRRig), "UpdateFriendshipBracelet")]
        public class BraceletPatch
        {
            public static bool enabled;

            public static bool Prefix(VRRig __instance) =>
                !enabled;
        }

        public static void SetBraceletState(bool enable, bool isLeftHand) =>
            GorillaTagger.Instance.myVRRig.SendRPC("EnableNonCosmeticHandItemRPC", RpcTarget.All, enable, isLeftHand);

        public static float isDirtyDelay;
        public static void RainbowBracelet()
        {
            BraceletPatch.enabled = true;
            if (!VRRig.LocalRig.nonCosmeticRightHandItem.IsEnabled)
            {
                SetBraceletState(true, false);
                RPCProtection();

                VRRig.LocalRig.nonCosmeticRightHandItem.EnableItem(true);
            }
            List<Color> rgbColors = new List<Color>();
            for (int i = 0; i < 10; i++)
                rgbColors.Add(Color.HSVToRGB((Time.frameCount / 180f + i / 10f) % 1f, 1f, 1f));

            VRRig.LocalRig.reliableState.isBraceletLeftHanded = false;
            VRRig.LocalRig.reliableState.braceletSelfIndex = 99;
            VRRig.LocalRig.reliableState.braceletBeadColors = rgbColors;
            VRRig.LocalRig.friendshipBraceletRightHand.UpdateBeads(rgbColors, 99);

            if (Time.time > isDirtyDelay)
            {
                isDirtyDelay = Time.time + 0.1f;
                VRRig.LocalRig.reliableState.SetIsDirty();
            }
        }

        public static GameObject RightHandBottom;
        public static GameObject RightHandTop;
        public static GameObject RightHandLeft;
        public static GameObject RightHandRight;
        public static GameObject RightHandFront;
        public static GameObject RightHandBack;

        public static GameObject LeftHandBottom;
        public static GameObject LeftHandTop;
        public static GameObject LeftHandLeft;
        public static GameObject LeftHandRight;
        public static GameObject LeftHandFront;
        public static GameObject LeftHandBack;
        private const float colorSpeed = 0.25f;
        // hue offsets for the six faces (spread across the spectrum)
        private static readonly float[] faceOffsets = new float[] { 0f, 0.1666667f, 0.3333333f, 0.5f, 0.6666667f, 0.8333333f };

        public static void StickyPlatforms()
        {
            if (ControllerInputPoller.instance.rightGrab && RightHandBottom == null)
            {
                RightHandBottom = GameObject.CreatePrimitive(PrimitiveType.Cube);
                RightHandBottom.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                RightHandBottom.transform.position = Main.TrueRightHand().position - new Vector3(0, 0.05f, 0);
                RightHandBottom.transform.rotation = Main.TrueRightHand().rotation;
                // ensure unique material instance
                var rBottom = RightHandBottom.GetComponent<Renderer>();
                rBottom.material.color = Color.HSVToRGB(faceOffsets[0], 1f, 1f);

                RightHandTop = GameObject.CreatePrimitive(PrimitiveType.Cube);
                RightHandTop.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                RightHandTop.transform.position = Main.TrueRightHand().position + new Vector3(0, 0.05f, 0);
                RightHandTop.transform.rotation = Main.TrueRightHand().rotation;
                RightHandTop.GetComponent<Renderer>().enabled = false;
                RightHandTop.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[1], 1f, 1f);

                RightHandRight = GameObject.CreatePrimitive(PrimitiveType.Cube);
                RightHandRight.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                RightHandRight.transform.position = Main.TrueRightHand().position + new Vector3(0.05f, 0, 0);
                RightHandRight.transform.eulerAngles = Main.TrueRightHand().rotation.eulerAngles + new Vector3(0, 0, 90);
                RightHandRight.GetComponent<Renderer>().enabled = false;
                RightHandRight.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[2], 1f, 1f);

                RightHandLeft = GameObject.CreatePrimitive(PrimitiveType.Cube);
                RightHandLeft.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                RightHandLeft.transform.position = Main.TrueRightHand().position - new Vector3(0.05f, 0, 0);
                RightHandLeft.transform.eulerAngles = Main.TrueRightHand().rotation.eulerAngles + new Vector3(0, 0, 90);
                RightHandLeft.GetComponent<Renderer>().enabled = false;
                RightHandLeft.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[3], 1f, 1f);

                RightHandFront = GameObject.CreatePrimitive(PrimitiveType.Cube);
                RightHandFront.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                RightHandFront.transform.position = Main.TrueRightHand().position + new Vector3(0, 0, 0.05f);
                RightHandFront.transform.eulerAngles = Main.TrueRightHand().rotation.eulerAngles + new Vector3(90, 0, 0);
                RightHandFront.GetComponent<Renderer>().enabled = false;
                RightHandFront.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[4], 1f, 1f);

                RightHandBack = GameObject.CreatePrimitive(PrimitiveType.Cube);
                RightHandBack.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                RightHandBack.transform.position = Main.TrueRightHand().position - new Vector3(0, 0, 0.05f);
                RightHandBack.transform.eulerAngles = Main.TrueRightHand().rotation.eulerAngles + new Vector3(90, 0, 0);
                RightHandBack.GetComponent<Renderer>().enabled = false;
                RightHandBack.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[5], 1f, 1f);
            }
            else if (!ControllerInputPoller.instance.rightGrab)
            {
                DestroyWithMaterial(RightHandBottom);
                DestroyWithMaterial(RightHandTop);
                DestroyWithMaterial(RightHandLeft);
                DestroyWithMaterial(RightHandRight);
                DestroyWithMaterial(RightHandFront);
                DestroyWithMaterial(RightHandBack);
            }

            if (ControllerInputPoller.instance.leftGrab && LeftHandBottom == null)
            {
                LeftHandBottom = GameObject.CreatePrimitive(PrimitiveType.Cube);
                LeftHandBottom.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                LeftHandBottom.transform.position = Main.TrueLeftHand().position - new Vector3(0, 0.05f, 0);
                LeftHandBottom.transform.rotation = Main.TrueLeftHand().rotation;
                LeftHandBottom.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[0], 1f, 1f);

                LeftHandTop = GameObject.CreatePrimitive(PrimitiveType.Cube);
                LeftHandTop.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                LeftHandTop.transform.position = Main.TrueLeftHand().position + new Vector3(0, 0.05f, 0);
                LeftHandTop.transform.rotation = Main.TrueLeftHand().rotation;
                LeftHandTop.GetComponent<Renderer>().enabled = false;
                LeftHandTop.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[1], 1f, 1f);

                LeftHandRight = GameObject.CreatePrimitive(PrimitiveType.Cube);
                LeftHandRight.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                LeftHandRight.transform.position = Main.TrueLeftHand().position + new Vector3(0.05f, 0, 0);
                LeftHandRight.transform.eulerAngles = Main.TrueLeftHand().rotation.eulerAngles + new Vector3(0, 0, 90);
                LeftHandRight.GetComponent<Renderer>().enabled = false;
                LeftHandRight.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[2], 1f, 1f);

                LeftHandLeft = GameObject.CreatePrimitive(PrimitiveType.Cube);
                LeftHandLeft.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                LeftHandLeft.transform.position = Main.TrueLeftHand().position - new Vector3(0.05f, 0, 0);
                LeftHandLeft.transform.eulerAngles = Main.TrueLeftHand().rotation.eulerAngles + new Vector3(0, 0, 90);
                LeftHandLeft.GetComponent<Renderer>().enabled = false;
                LeftHandLeft.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[3], 1f, 1f);

                LeftHandFront = GameObject.CreatePrimitive(PrimitiveType.Cube);
                LeftHandFront.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                LeftHandFront.transform.position = Main.TrueLeftHand().position + new Vector3(0, 0, 0.05f);
                LeftHandFront.transform.eulerAngles = Main.TrueLeftHand().rotation.eulerAngles + new Vector3(90, 0, 0);
                LeftHandFront.GetComponent<Renderer>().enabled = false;
                LeftHandFront.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[4], 1f, 1f);

                LeftHandBack = GameObject.CreatePrimitive(PrimitiveType.Cube);
                LeftHandBack.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                LeftHandBack.transform.position = Main.TrueLeftHand().position - new Vector3(0, 0, 0.05f);
                LeftHandBack.transform.eulerAngles = Main.TrueLeftHand().rotation.eulerAngles + new Vector3(90, 0, 0);
                LeftHandBack.GetComponent<Renderer>().enabled = false;
                LeftHandBack.GetComponent<Renderer>().material.color = Color.HSVToRGB(faceOffsets[5], 1f, 1f);
            }
            else if (!ControllerInputPoller.instance.leftGrab)
            {
                DestroyWithMaterial(LeftHandBottom);
                DestroyWithMaterial(LeftHandTop);
                DestroyWithMaterial(LeftHandLeft);
                DestroyWithMaterial(LeftHandRight);
                DestroyWithMaterial(LeftHandFront);
                DestroyWithMaterial(LeftHandBack);
            }

            // Update colors each frame for existing objects (even if renderer disabled we update the material color so when enabled it shows current color)
            UpdateColorCycle(RightHandBottom, faceOffsets[0]);
            UpdateColorCycle(RightHandTop, faceOffsets[1]);
            UpdateColorCycle(RightHandLeft, faceOffsets[3]);
            UpdateColorCycle(RightHandRight, faceOffsets[2]);
            UpdateColorCycle(RightHandFront, faceOffsets[4]);
            UpdateColorCycle(RightHandBack, faceOffsets[5]);

            UpdateColorCycle(LeftHandBottom, faceOffsets[0]);
            UpdateColorCycle(LeftHandTop, faceOffsets[1]);
            UpdateColorCycle(LeftHandLeft, faceOffsets[3]);
            UpdateColorCycle(LeftHandRight, faceOffsets[2]);
            UpdateColorCycle(LeftHandFront, faceOffsets[4]);
            UpdateColorCycle(LeftHandBack, faceOffsets[5]);
        }



        private static void UpdateColorCycle(GameObject obj, float offset)
        {
            if (obj == null) return;
            var rend = obj.GetComponent<Renderer>();
            if (rend == null) return;
            // ensure material instance exists (renderer.material will create one if needed)
            var mat = rend.material;
            if (mat == null) return;
            float hue = (Time.time * colorSpeed + offset) % 1f;
            mat.color = Color.HSVToRGB(hue, 1f, 1f);
        }

        private static void DestroyWithMaterial(GameObject obj)
        {
            if (obj == null) return;
            var rend = obj.GetComponent<Renderer>();
            if (rend != null)
            {
                // Destroy material instance to avoid leaks (use DestroyImmediate only in editor; here use Destroy)
                var mat = rend.material;
                if (mat != null)
                {
                    GameObject.Destroy(mat, Time.deltaTime);
                }
            }
            GameObject.Destroy(obj, Time.deltaTime);
        }

        public static void RemoveRainbowBracelet()
        {
            BraceletPatch.enabled = false;
            if (!VRRig.LocalRig.nonCosmeticRightHandItem.IsEnabled)
            {
                SetBraceletState(false, false);
                RPCProtection();

                VRRig.LocalRig.nonCosmeticRightHandItem.EnableItem(false);
            }

            VRRig.LocalRig.reliableState.isBraceletLeftHanded = false;
            VRRig.LocalRig.reliableState.braceletSelfIndex = 0;
            VRRig.LocalRig.reliableState.braceletBeadColors.Clear();
            VRRig.LocalRig.UpdateFriendshipBracelet();

            VRRig.LocalRig.reliableState.SetIsDirty();
        }

        public static void GiveBuilderWatch()
        {
            VRRig.LocalRig.EnableBuilderResizeWatch(true);
            RPCProtection();
        }

        public static void RemoveBuilderWatch()
        {
            VRRig.LocalRig.EnableBuilderResizeWatch(false);
            RPCProtection();
        }


        public static void RPCProtection()
        {
            if (!PhotonNetwork.InRoom)
                return;

            try
            {
                var gorillaIOBT = UnityEngine.Object.FindFirstObjectByType<GorillaIOBT>();
                if (gorillaIOBT != null)
                {
                    // gorillaIOBT.rpcErrorMax = int.MaxValue;
                }
                else
                {
                    NotifiLib.SendNotification("GorillaIOBT instance not found.");
                }

                PhotonNetwork.MaxResendsBeforeDisconnect = int.MaxValue;
                PhotonNetwork.QuickResends = int.MaxValue;

                PhotonNetwork.SendAllOutgoingCommands();
            }
            catch { NotifiLib.SendNotification("RPC protection failed, are you in a lobby?"); }
        }


        public static void WASDFly()
        {
            GTPlayer.Instance.GetComponent<Rigidbody>().linearVelocity = new Vector3(0f, 0.067f, 0f);
            if (Mouse.current.rightButton.isPressed)
            {
                Vector3 eulerAngles = GTPlayer.Instance.RightHand.controllerTransform.parent.rotation.eulerAngles;
                eulerAngles.y += Mouse.current.delta.x.ReadValue() / (float)Screen.width * 250f;
                eulerAngles.x -= Mouse.current.delta.y.ReadValue() / (float)Screen.height * 250f;
                GTPlayer.Instance.RightHand.controllerTransform.parent.rotation = Quaternion.Euler(eulerAngles);
            }
            float speed = 7.5f;
            if (UnityInput.Current.GetKey(KeyCode.LeftShift)) speed = 15f;
            if (UnityInput.Current.GetKey(KeyCode.W))
            {
                GorillaTagger.Instance.rigidbody.transform.position += GorillaLocomotion.GTPlayer.Instance.RightHand.controllerTransform.parent.forward * Time.deltaTime * speed;
            }
            if (UnityInput.Current.GetKey(KeyCode.S))
            {
                GorillaTagger.Instance.rigidbody.transform.position += GorillaLocomotion.GTPlayer.Instance.RightHand.controllerTransform.parent.forward * Time.deltaTime * -speed;
            }
            if (UnityInput.Current.GetKey(KeyCode.A))
            {
                GorillaTagger.Instance.rigidbody.transform.position += GorillaLocomotion.GTPlayer.Instance.RightHand.controllerTransform.parent.right * Time.deltaTime * -speed;
            }
            if (UnityInput.Current.GetKey(KeyCode.D))
            {
                GorillaTagger.Instance.rigidbody.transform.position += GorillaLocomotion.GTPlayer.Instance.RightHand.controllerTransform.parent.right * Time.deltaTime * speed;
            }
            if (UnityInput.Current.GetKey(KeyCode.Space))
            {
                GorillaTagger.Instance.rigidbody.transform.position += new Vector3(0f, Time.deltaTime * speed, 0f);
            }
            if (UnityInput.Current.GetKey(KeyCode.LeftControl))
            {
                GorillaTagger.Instance.rigidbody.transform.position += new Vector3(0f, Time.deltaTime * -speed, 0f);
            }
        }

        private static Vector3? oldLocalPosition;
        public static void PCButtonClick()
        {
            if (Mouse.current.leftButton.isPressed)
            {
                Ray ray = TPC.ScreenPointToRay(Mouse.current.position.ReadValue());
                Physics.Raycast(ray, out var Ray, 512f, NoInvisLayerMask());

                oldLocalPosition ??= GorillaTagger.Instance.rightHandTriggerCollider.transform.localPosition;
                GorillaTagger.Instance.rightHandTriggerCollider.GetComponent<TransformFollow>().enabled = false;
                GorillaTagger.Instance.rightHandTriggerCollider.transform.position = Ray.point;
            }
            else
            {
                if (oldLocalPosition != null)
                {
                    GorillaTagger.Instance.rightHandTriggerCollider.transform.localPosition = oldLocalPosition.Value;
                    oldLocalPosition = null;
                }
                GorillaTagger.Instance.rightHandTriggerCollider.GetComponent<TransformFollow>().enabled = true;
            }
        }

        public static void DisablePCButtonClick()
        {
            if (oldLocalPosition != null)
            {
                GorillaTagger.Instance.rightHandTriggerCollider.transform.localPosition = oldLocalPosition.Value;
                oldLocalPosition = null;
            }
        }
        public static void speedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 11f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 1.3f;
        }
        public static void zerogravity()
        {
            GorillaLocomotion.GTPlayer.Instance.bodyCollider.attachedRigidbody.AddForce(UnityEngine.Vector3.up * 9.82f, ForceMode.Acceleration);
        }

        public static void lowgravity()
        {
            GorillaLocomotion.GTPlayer.Instance.bodyCollider.attachedRigidbody.AddForce(UnityEngine.Vector3.up * 6.67f, UnityEngine.ForceMode.Acceleration);
        }

        public static void highgravity()
        {
            GorillaLocomotion.GTPlayer.Instance.bodyCollider.attachedRigidbody.AddForce(UnityEngine.Vector3.down * 7.79f, UnityEngine.ForceMode.Acceleration);
        }

        public static void HandFly()
        {
            if (ControllerInputPoller.instance.rightControllerPrimaryButton)
            {
                GTPlayer.Instance.transform.position += GorillaTagger.Instance.rightHandTransform.transform.forward * (Time.deltaTime * 10);
                GorillaTagger.Instance.rigidbody.linearVelocity = Vector3.zero;
            }
        }
        private static float sizescale = 1f;
        public static void sizechanger()
        {
            if (ControllerInputPoller.instance.rightControllerPrimaryButton)
            {
                sizescale = 1f;
            }

            if (ControllerInputPoller.instance.leftControllerIndexFloat > 0.1f)
            {
                sizescale -= 0.05f;
            }

            if (ControllerInputPoller.instance.rightControllerIndexFloat > 0.1f)
            {
                sizescale += 0.05f;
            }
            if (sizescale <= 0f)
            {
                sizescale = 0.05f;
            }
            GTPlayer.Instance.scaleMultiplier = sizescale;
        }

        private static Quaternion grabHeadRot;

        private static Vector3 grabLeftHandPos;
        private static Quaternion grabLeftHandRot;

        private static Vector3 grabRightHandPos;
        private static Quaternion grabRightHandRot;

        public static void GrabRig()
        {
            if (ControllerInputPoller.instance.rightGrab)
            {
                if (grabHeadRot == Quaternion.identity)
                    grabHeadRot = VRRig.LocalRig.transform.InverseTransformRotation(VRRig.LocalRig.head.rigTarget.transform.rotation);

                if (grabLeftHandPos == Vector3.zero)
                    grabLeftHandPos = VRRig.LocalRig.transform.InverseTransformPoint(VRRig.LocalRig.leftHand.rigTarget.transform.position);

                if (grabLeftHandRot == Quaternion.identity)
                    grabLeftHandRot = VRRig.LocalRig.transform.InverseTransformRotation(VRRig.LocalRig.leftHand.rigTarget.transform.rotation);

                if (grabRightHandPos == Vector3.zero)
                    grabRightHandPos = VRRig.LocalRig.transform.InverseTransformPoint(VRRig.LocalRig.rightHand.rigTarget.transform.position);

                if (grabRightHandRot == Quaternion.identity)
                    grabRightHandRot = VRRig.LocalRig.transform.InverseTransformRotation(VRRig.LocalRig.rightHand.rigTarget.transform.rotation);

                VRRig.LocalRig.enabled = false;

                VRRig.LocalRig.transform.position = GorillaTagger.Instance.rightHandTransform.position;
                VRRig.LocalRig.transform.rotation = Quaternion.Euler(new Vector3(0f, GorillaTagger.Instance.rightHandTransform.rotation.eulerAngles.y, 0f));

                VRRig.LocalRig.head.rigTarget.transform.rotation = GorillaTagger.Instance.rightHandTransform.TransformRotation(grabHeadRot);

                VRRig.LocalRig.leftHand.rigTarget.transform.position = GorillaTagger.Instance.rightHandTransform.TransformPoint(grabLeftHandPos);
                VRRig.LocalRig.leftHand.rigTarget.transform.rotation = GorillaTagger.Instance.rightHandTransform.TransformRotation(grabLeftHandRot);

                VRRig.LocalRig.rightHand.rigTarget.transform.position = GorillaTagger.Instance.rightHandTransform.TransformPoint(grabRightHandPos);
                VRRig.LocalRig.rightHand.rigTarget.transform.rotation = GorillaTagger.Instance.rightHandTransform.TransformRotation(grabRightHandRot);
            }
            else
            {
                VRRig.LocalRig.enabled = true;

                grabHeadRot = Quaternion.identity;

                grabLeftHandPos = Vector3.zero;
                grabRightHandPos = Vector3.zero;

                grabLeftHandRot = Quaternion.identity;
                grabRightHandRot = Quaternion.identity;
            }
        }
        public static void NoClip()
        {
            if (ControllerInputPoller.instance.rightControllerTriggerButton)
            {
                MeshCollider[] array = Resources.FindObjectsOfTypeAll<MeshCollider>();
                for (int i = 0; i < array.Length; i++)
                {
                    array[i].enabled = false;
                }
            }
            else
            {
                MeshCollider[] array = Resources.FindObjectsOfTypeAll<MeshCollider>();
                for (int i = 0; i < array.Length; i++)
                {
                    array[i].enabled = true;
                }
            }
        }
        public static void mosaspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 10f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 1.1f;
        }
        public static void superspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 20f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 1.9f;
        }
        public static void tttspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 9f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 1.1f;
        }
        public static void megaspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 20f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 2.2f;
        }
        public static void hugespeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 35f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 2.9f;
        }
        public static void bigspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 42f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 3.0f;
        }
        public static void giantspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 50f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 3.6f;
        }
        public static void jalaxyspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 100f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 5.7f;
        }
        public static void undetectablespeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 10f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 1.1f;
        }
        public static void setprivate()
        {

            PhotonNetwork.CurrentRoom.IsVisible = false;
        }
        public static void setpublic()
        {
            PhotonNetwork.CurrentRoom.IsVisible = true;
        }
        public static void Frozone()
        {
            if (ControllerInputPoller.instance.rightGrab)
            {
                GameObject FrozoneCube = GameObject.CreatePrimitive(PrimitiveType.Cube);
                FrozoneCube.AddComponent<GorillaSurfaceOverride>().overrideIndex = 61;
                FrozoneCube.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                FrozoneCube.transform.position = TrueRightHand().position - new Vector3(0, .05f, 0);
                FrozoneCube.transform.rotation = TrueRightHand().rotation;

                FrozoneCube.GetComponent<Renderer>().material.color = Color.black;
                GameObject.Destroy(FrozoneCube, 5);
            }

            if (ControllerInputPoller.instance.leftGrab)
            {
                GameObject FrozoneCube = GameObject.CreatePrimitive(PrimitiveType.Cube);
                FrozoneCube.AddComponent<GorillaSurfaceOverride>().overrideIndex = 61;
                FrozoneCube.transform.localScale = new Vector3(0.025f, 0.3f, 0.4f);
                FrozoneCube.transform.position = TrueLeftHand().position - new Vector3(0, .05f, 0);
                FrozoneCube.transform.rotation = TrueLeftHand().rotation;

                FrozoneCube.GetComponent<Renderer>().material.color = Color.white;
                GameObject.Destroy(FrozoneCube, 5);
            }
        }
        public static void DisableNetworkTriggers()
        {
            GameObject.Find("Environment Objects/TriggerZones_Prefab/JoinRoomTriggers_Prefab").SetActive(false);
        }
        public static void rightgripspeedboost()
        {
            if (ControllerInputPoller.instance.leftGrab)
            {
                GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 9f;
            }
        }
        public static void RightGripFly()
        {
            if (ControllerInputPoller.instance.rightGrab)
            {
                GorillaLocomotion.GTPlayer.Instance.transform.position += (GorillaLocomotion.GTPlayer.Instance.headCollider.transform.forward * Time.deltaTime) * 15;
                GorillaLocomotion.GTPlayer.Instance.GetComponent<Rigidbody>().velocity = Vector3.zero;
            }
        }
        public static void iiDkspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 1000f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 7.6f;
        }
        public static bool IsGhost;
        public static bool HasClicked;
        public static void GhostMonke()
        {
            if (ControllerInputPoller.instance.rightControllerPrimaryButton || Mouse.current.leftButton.isPressed)
            {
                if (!HasClicked)
                {
                    IsGhost = !IsGhost;
                    HasClicked = true;
                }
            }
            else if (!ControllerInputPoller.instance.rightControllerPrimaryButton && !Mouse.current.leftButton.isPressed)
                HasClicked = false;

            if (IsGhost)
                GorillaTagger.Instance.offlineVRRig.enabled = false;
            else
                GorillaTagger.Instance.offlineVRRig.enabled = true;
        }
        public static void UpAndDown()
        {
            if (ControllerInputPoller.instance.rightControllerIndexFloat > 0.4f)
            {
                GorillaTagger.Instance.rigidbody.AddForce(GorillaTagger.Instance.offlineVRRig.transform.up * 5000f);
            }
            else if (ControllerInputPoller.instance.leftControllerIndexFloat > 0.4f)
            {
                GorillaTagger.Instance.rigidbody.AddForce(GorillaTagger.Instance.offlineVRRig.transform.up * -5000f);
            }
        }
        public static void PBKillAll()
        {
            if (!PhotonNetwork.IsMasterClient || GorillaPaintbrawlManager.instance == null) return;

            foreach (var vrrig in GorillaParent.instance.vrrigs)
            {
                NetPlayer player = vrrig.OwningNetPlayer;
                GorillaPaintbrawlManager.instance.HitPlayer(player);
            }
        }
        public static void TracerV2()
        {
            foreach (VRRig vrrig in GorillaParent.instance.vrrigs)
            {
                if (vrrig != GorillaTagger.Instance.offlineVRRig)
                {
                    GameObject line = new GameObject("Line");
                    LineRenderer liner = line.AddComponent<LineRenderer>();
                    UnityEngine.Color thecolor = vrrig.playerColor;
                    liner.startColor = thecolor; liner.endColor = thecolor; liner.startWidth = 0.010f; liner.endWidth = 0.010f; liner.positionCount = 2; liner.useWorldSpace = true;
                    liner.SetPosition(0, GorillaTagger.Instance.rightHandTransform.position);
                    liner.SetPosition(1, vrrig.transform.position);
                    liner.material.shader = Shader.Find("GUI/Text Shader");
                    UnityEngine.Object.Destroy(line, Time.deltaTime);
                }
            }
        }
        private static float flapTime;
        public static void BirdFly()
        {
            if (Vector3.Distance(GorillaTagger.Instance.leftHandTransform.position, GorillaTagger.Instance.headCollider.transform.position) < 0.63f || Vector3.Distance(GorillaTagger.Instance.rightHandTransform.position, GorillaTagger.Instance.headCollider.transform.position) < 0.63f)
                return;

            if (Vector3.Distance(GorillaTagger.Instance.leftHandTransform.position, GorillaTagger.Instance.rightHandTransform.position) < 1f)
                return;
            if (Physics.Raycast(GorillaTagger.Instance.bodyCollider.attachedRigidbody.position, Vector3.down, hitInfo: out _, Physics.AllLayers))
                return;

            UnityEngine.XR.InputDevice LeftHand = InputDevices.GetDeviceAtXRNode(XRNode.LeftHand);
            UnityEngine.XR.InputDevice RightHand = InputDevices.GetDeviceAtXRNode(XRNode.RightHand);

            if (LeftHand.TryGetFeatureValue(UnityEngine.XR.CommonUsages.deviceVelocity, out Vector3 leftVel) && RightHand.TryGetFeatureValue(UnityEngine.XR.CommonUsages.deviceVelocity, out Vector3 rightVel))
            {
                if (Time.time - flapTime < 0.4f) return;

                if (leftVel.y < -1.2f && rightVel.y < -1.2f)
                {
                    float force = Mathf.Min(6f * ((Mathf.Abs(leftVel.y) + Mathf.Abs(rightVel.y)) / 2f) / 1.2f, 9f);
                    GorillaTagger.Instance.bodyCollider.attachedRigidbody.AddForce(Vector3.up * force, ForceMode.VelocityChange);

                    flapTime = Time.time;
                }
            }
        }
        public static void ForceTagFreeze() =>
        GTPlayer.Instance.disableMovement = true;
        public static void NoTagFreeze() =>
                GTPlayer.Instance.disableMovement = false;
        public static void clonexspeedboost()
        {
            GorillaLocomotion.GTPlayer.Instance.maxJumpSpeed = 8f;
            GorillaLocomotion.GTPlayer.Instance.jumpMultiplier = 1.2f;
        }
        public static void triggerplats()
        {
            if (ControllerInputPoller.instance.leftControllerTriggerButton)
            {
                if (PlatL == null)
                {
                    PlatL = GameObject.CreatePrimitive(PrimitiveType.Cube);
                    PlatL.transform.localScale = new Vector3(0.04f, 0.28f, 0.35f);
                    PlatL.transform.position = GorillaTagger.Instance.leftHandTransform.position + new Vector3(0f, -0.06f, 0f);
                    PlatL.transform.rotation = GorillaTagger.Instance.leftHandTransform.rotation;
                    PlatL.GetComponent<Renderer>().material.shader = Shader.Find("GorillaTag/UberShader");
                    PlatL.GetComponent<Renderer>().material.color = StupidTemplate.Settings.backgroundColor.GetCurrentColor();
                    ColorChanger.cc = PlatL.AddComponent<ColorChanger>(); // 
                }
            }
            else
            {
                if (PlatL != null)
                {
                    Rigidbody rb = PlatL.AddComponent(typeof(Rigidbody)) as Rigidbody;
                    rb.velocity = GorillaLocomotion.GTPlayer.Instance.GetHandVelocityTracker(true).GetAverageVelocity(true, 0);
                    GameObject.Destroy(PlatL, 0.1f);
                    PlatL = null;
                }
            }

            if (ControllerInputPoller.instance.rightControllerTriggerButton)
            {
                if (PlatR == null)
                {
                    PlatR = GameObject.CreatePrimitive(PrimitiveType.Cube);
                    PlatR.transform.localScale = new Vector3(0.04f, 0.28f, 0.35f);
                    PlatR.transform.position = GorillaTagger.Instance.rightHandTransform.position + new Vector3(0f, -0.06f, 0f);
                    PlatR.transform.rotation = GorillaTagger.Instance.rightHandTransform.rotation;
                    PlatR.GetComponent<Renderer>().material.shader = Shader.Find("GorillaTag/UberShader");
                    PlatR.GetComponent<Renderer>().material.color = StupidTemplate.Settings.backgroundColor.GetCurrentColor();
                    ColorChanger.cc = PlatR.AddComponent<ColorChanger>();
                }
            }
            else
            {
                if (PlatL != null)
                {
                    Rigidbody rb = PlatR.AddComponent(typeof(Rigidbody)) as Rigidbody;
                    rb.velocity = GorillaLocomotion.GTPlayer.Instance.GetHandVelocityTracker(true).GetAverageVelocity(true, 0);
                    GameObject.Destroy(PlatR, 0.1f);
                    PlatR = null;
                }
            }
        }
        public static void KickStump()
        {
            Dictionary<byte, object> dictionary = new Dictionary<byte, object>();
            Dictionary<byte, object> dictionary2 = dictionary;
            byte key = 251;
            Hashtable hashtable = new Hashtable();
            hashtable.Add("gameMode", true);
            dictionary2.Add(key, hashtable);
            dictionary.Add(250, true);
            PhotonNetwork.NetworkingClient.LoadBalancingPeer.SendOperation(252, dictionary, SendOptions.SendReliable);
        }
        public static void EnableLucy()
        {
            bool flag = !GameObject.Find("Environment Objects/05Maze_PersistentObjects/MinesSecondLookSkeleton/Offset/SpookySkeletonParent").activeSelf;
            if (flag)
            {
                GameObject.Find("Environment Objects/05Maze_PersistentObjects/MinesSecondLookSkeleton").GetComponent<SecondLookSkeletonSynchValues>().SendRPC("RemoteActivateGhost", 2, Array.Empty<object>());
            }
        }
        public static void ReallyLongArms()
        {
            GTPlayer.Instance.transform.localScale = new Vector3(2f, 2f, 2f);
        }
    }
}
