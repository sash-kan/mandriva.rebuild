export LC_ALL = C
mirror = /mnt/mirrors/mandriva/official/2011
user = mandriva
withouturpmiroot = 1
withouturpmirootu = 0
v = 2

arch := $(shell cat /etc/product.id | grep -Eo 'arch=[^,$$]+' | sed 's/arch=//')
srpms = $(shell cat srpms.list)

cachedir = cache
chrootdir = chroot
chroottardir = chroottars
reportdir = report
failedlogdir = failedlogs
chroottarsarchdir = $(foreach a,$(arch),$(chroottardir)/$(a))
chroottars = $(foreach a,$(arch),$(chroottardir)/$(a).tar)
reportpackages = $(foreach p,$(addprefix $(reportdir)/,$(srpms)),$(foreach a,$(arch),$(p).$(a)))

srpmsincache = $(addprefix $(cachedir)/,$(srpms))

rebuildeddir = rebuilded
rebuildedpackages = $(addprefix $(rebuildeddir)/,$(patchedsrpms))
patchedsrpms = $(basename $(shell ls patches))
patchedsrpmsincache = $(addprefix $(cachedir)/,$(patchedsrpms))
rebuildchrootdir = rebuildchroot

repos1 = main contrib non-free
repos2 = release updates
repos3 = $(foreach r1,$(repos1),$(foreach r2,$(repos2),$(r1)/$(r2)))
repos  = $(foreach r,$(repos3),$(foreach p,SRPMS $(archat)/media,$(p)/$(r)))

repos.add.i586 =
repos.add.x86_64 = i586/media/main/release i586/media/main/updates

percent = %
archat = $(strip $(foreach a,$(arch),$(findstring $(a),$@)))
pkgat = $(basename $(notdir $@))
pkgat_mdv2000 = $(shell echo "$(basename $(notdir $@))" | sed -r 's/mdv201[01]\.[01]//')
ifeq ($(v),0)
	output = &>/dev/null
	at = @
else
ifeq ($(v),1)
	output = &>/dev/null
	at =
else
	output =
	at =
endif
endif

ifeq ($(withouturpmiroot),1)
	withoutat =
	withoutcr =
else
	withoutat = --root $@
	withoutcr = --root $(chrootdir)
endif

ifeq ($(withouturpmirootu),1)
	withoutatu =
	withoutcru =
else
	withoutatu = --urpmi-root $@
	withoutcru = --urpmi-root $(chrootdir)
endif

.SECONDEXPANSION:

all: srpms.list $(chroottardir) $(cachedir) $(reportdir) $(failedlogdir) $(chroottars) $(srpmsincache) $(reportpackages)

justrebuild: $(rebuildeddir) $(chroottardir) $(cachedir) $$(word 1,$$(chroottars)) $(patchedsrpmsincache) $(rebuildedpackages) delete.rebuildchrootdir

$(rebuildedpackages): $(rebuildchrootdir)
	$(at)# rebuildedpackages
	$(at)# $@=rebuilded/timezone-2011k-1.1.src.rpm $(notdir $@)=timezone-2011k-1.1.src.rpm $(cachedir)/$(notdir $@)=cache/timezone-2011k-1.1.src.rpm
	$(at)# copy package to /tmp
	$(at)sudo cp $(cachedir)/$(notdir $@) $(rebuildchrootdir)/tmp $(output)
	$(at)# install file
	$(at)sudo chroot $(rebuildchrootdir) su - $(user) -c '/bin/rpm --nodeps -i /tmp/$(notdir $@)' $(output)
	$(at)# apply patch if exists
	$(at)if [ -f "patches/$(notdir $@).patch" ] ; then \
		sudo cp "patches/$(notdir $@).patch" $(rebuildchrootdir)/tmp; \
		sudo chroot $(rebuildchrootdir) su - $(user) -c 'patch -d ~/rpmbuild -p0 -i /tmp/$(notdir $@).patch'; \
		else :; fi $(output)
	$(at)# rpmbuild
	$(at)-sudo chroot $(rebuildchrootdir) su - $(user) -c '/usr/bin/rpmbuild -bs rpmbuild/SPECS/*.spec' \
		2>&1 | tee -a .log $(output)
	$(at)cp $(rebuildchrootdir)/home/$(user)/rpmbuild/SRPMS/* $(rebuildeddir) $(output)
	$(at)-sudo chroot $(rebuildchrootdir) su - $(user) -c 'rm -rf ~/rpmbuild' $(output)
	$(at)-sudo rm $(rebuildchrootdir)/tmp/$(notdir $@)* $(output)

$(rebuildchrootdir):
	echo $@
	mkdir -p $@
	$(at)[ -d $@/tmp ] || sudo tar -xf $(word 1,$(chroottars)) -C $@ $(output)
	$(at)# create user
	$(at)sudo chroot $@ adduser $(user) $(output)

srpms.list:
	touch $@
	$(warning create empty $@)

# build initial chroot
$(chroottarsarchdir): 
	$(at)# umount sys+proc
	$(at)-[ -f .procmounted.$(archat) ] && sudo umount -lf $@/proc; : $(output)
	$(at)-[ -f .sysmounted.$(archat) ] && sudo umount -lf $@/sys; : $(output)
	$(at)-rm -f .procmounted.$(archat) .sysmounted.$(archat) $(output)
	$(at)for i in $(repos) $(repos.add.$(archat)); do \
		sudo urpmi.addmedia $(withoutatu) $$(echo $$i | sed 's!/!_!g') \
			$(mirror)/$$i; \
	done $(output)
	$(at)sudo urpmi  --no-suggests --excludedocs --no-verify-rpm --auto $(withoutat) \
		$(withoutatu) basesystem-minimal $(output)
	$(at)# mount sys+proc
	$(at)[ ! -f .procmounted.$(archat) ] && sudo mount -o bind /proc $@/proc && \
		touch .procmounted.$(archat) $(output)
	$(at)[ ! -f .sysmounted.$(archat) ] && sudo mount -o bind /sys $@/sys && \
		touch .sysmounted.$(archat) $(output)
	$(at)sudo urpmi  --no-suggests --excludedocs --no-verify-rpm --auto $(withoutat) \
		$(withoutatu) shadow-utils rpm tar rpm-build \
		rpm-mandriva-setup urpmi rsync bzip2 shadow-utils locales-en $(output)
	$(at)sudo chroot $@ sh -c 'cd /var/lib/rpm && db51_recover && rpm --rebuilddb' $(output)
	$(at)# umount sys+proc
	$(at)-[ -f .procmounted.$(archat) ] && sudo umount -lf $@/proc; : $(output)
	$(at)-[ -f .sysmounted.$(archat) ] && sudo umount -lf $@/sys; : $(output)
	$(at)-rm -f .procmounted.$(archat) .sysmounted.$(archat) $(output)

# make tarball with initial chroot
$(chroottars): $(chroottarsarchdir)
	$(at)-sudo tar --ignore-failed-read -cf $@ -C $(foreach d,$^,$(findstring $(d),$@)) . $(output)

# untar new chroot, build next package
$(reportpackages):
	$(at)# $@ p=$(pkgat)
	$(at)# umount sys+proc
	$(at)-[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys; : $(output)
	$(at)-[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc; : $(output)
	$(at)-[ -f .devptsmounted ] && sudo umount -lf $(chrootdir)/dev/pts; : $(output)
	$(at)-[ -f .devmounted ] && sudo umount -lf $(chrootdir)/dev; : $(output)
	$(at)-[ -f .mirmounted ] && sudo umount -lf $(chrootdir)/$(mirror); : $(output)
	$(at)-rm -f .sysmounted .procmounted .devmounted .devptsmounted .mirmounted $(output)
	$(at)# remove chroot
	$(at)-sudo rm -rf $(chrootdir) $(output)
	$(at)# untar chroot
	$(at)mkdir $(chrootdir) $(output)
	$(at)sudo tar -xf $(chroottardir)/$(archat).tar -C $(chrootdir) $(output)
	$(at)sudo mkdir -p $(chrootdir)$(mirror)
	$(at)# mount sys+proc
	$(at)[ ! -f .devmounted ] && sudo mount -o bind /dev $(chrootdir)/dev && touch .devmounted $(output)
	$(at)[ ! -f .devptsmounted ] && sudo mount -o bind /dev/pts $(chrootdir)/dev/pts && touch .devptsmounted $(output)
	$(at)[ ! -f .procmounted ] && sudo mount -o bind /proc $(chrootdir)/proc && touch .procmounted $(output)
	$(at)[ ! -f .sysmounted ] && sudo mount -o bind /sys $(chrootdir)/sys && touch .sysmounted $(output)
	$(at)[ ! -f .mirmounted ] && sudo mount -o bind $(mirror) $(chrootdir)/$(mirror) && touch .mirmounted $(output)
	$(at)# copy package to /tmp
	$(at)sudo cp -P $(cachedir)/$(pkgat) $(chrootdir)/tmp $(output)
	$(at)# create user
	$(at)sudo chroot $(chrootdir) adduser $(user) $(output)
	$(at)# install file
	$(at)sudo chroot $(chrootdir) su - $(user) -c '/bin/rpm --nodeps -i /tmp/$(pkgat)' $(output)
	$(at)# apply patch if exists
	$(at)if [ -f "patches/$(pkgat).patch" ] ; then \
		sudo cp "patches/$(pkgat).patch" $(chrootdir)/tmp; \
		sudo chroot $(chrootdir) su - $(user) -c 'patch -d ~/rpmbuild -p0 -i /tmp/$(pkgat).patch'; \
		else :; fi $(output)
	$(at)# install buildreqs
	$(at)# if the patch fixes builddeps, then we should try to install them after applying
	$(at)sudo chroot $(chrootdir) urpmi --noclean --no-suggests --excludedocs --no-verify-rpm \
		--auto $$(ls $(chrootdir)/home/$(user)/rpmbuild/SPECS/*.spec | sed 's/^$(chrootdir)//') \
		2>&1 | tee .log;
	$(at)# rpmbuild
	$(at)-sudo chroot $(chrootdir) su - $(user) -c '/usr/bin/rpmbuild -ba rpmbuild/SPECS/*.spec \
		--target=$(archat)'  2>&1 | tee -a .log $(output)
	$(at)# check if srpm was built. if failed, keep log
	$(at)if [ ! -f $(chrootdir)/home/$(user)/rpmbuild/SRPMS/$(pkgat_mdv2000) ] ; then \
		cp .log $(failedlogdir)/$(notdir $@).$$(date +"%Y%m%d.%H%M%S.%N").log; \
		echo "no srpm" > $@; \
	else \
		echo ok > $@; \
	fi $(output)
	$(at)rm -f .log $(output)
	$(at)touch $@ $(output)
	$(at)# umount sys+proc
	$(at)-[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys; : $(output)
	$(at)-[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc; : $(output)
	$(at)-[ -f .devptsmounted ] && sudo umount -lf $(chrootdir)/dev/pts; : $(output)
	$(at)-[ -f .devmounted ] && sudo umount -lf $(chrootdir)/dev; : $(output)
	$(at)-[ -f .mirmounted ] && sudo umount -lf $(chrootdir)/$(mirror); : $(output)
	$(at)-rm -f .sysmounted .procmounted .devmounted .devptsmounted .mirmounted $(output)

$(srpmsincache) $(patchedsrpmsincache):
	$(at)if [ -f "diversion/$(notdir $@)" ] ; \
	then d=$$(cat "diversion/$(notdir $@)"); \
	else d=$(notdir $@); fi; \
	p=$$(find $(mirror)/SRPMS -name $$d 2>.find.error | head -n 1); \
		if [ -z "$$p" ] ; then \
			for i in $(arch); do (echo "not in repo: "; cat .find.error) \
				> $(reportdir)/$(notdir $@).$$i; \
			done; \
		else ln -s $$p $@; fi $(output)
	$(at)-rm -f .find.error $(output)

$(cachedir) $(reportdir) $(chroottardir) $(failedlogdir) $(rebuildeddir):
	$(at)mkdir -p $@ $(output)

clearall: delete.cachedir delete.reportdir delete.chroottardir delete.chrootdir delete.failedlogdir delete.rebuildchrootdir

delete.%:
	$(at)-sudo rm -rf $($*) $(output)

delete.chrootdir:
	$(at)-[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys; : $(output)
	$(at)-[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc; : $(output)
	$(at)-[ -f .devptsmounted ] && sudo umount -lf $(chrootdir)/dev/pts; : $(output)
	$(at)-[ -f .devmounted ] && sudo umount -lf $(chrootdir)/dev; : $(output)
	$(at)-[ -f .mirmounted ] && sudo umount -lf $(chrootdir)/$(mirror); : $(output)
	$(at)-rm -f .sysmounted .procmounted .devmounted .devptsmounted .mirmounted $(output)
	$(at)-sudo rm -rf $(chrootdir) $(output)

delete.chroottardir:
	$(at)-for a in $(arch); do \
		[ -f .procmounted.$$a ] && sudo umount -lf $(chroottardir)/$$a/proc; \
		[ -f .sysmounted.$$a ] && sudo umount -lf $(chroottardir)/$$a/sys; \
		rm -f .procmounted.$$a .sysmounted.$$a; \
		sudo rm -rf $(chroottardir); \
	done $(output)
